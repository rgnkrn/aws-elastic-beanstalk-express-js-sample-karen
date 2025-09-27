pipeline {
    agent any   // Use any available Jenkins agent (no docker plugin needed)

    environment {
        REGISTRY    = "rgnkrn1234"   // e.g., "erginakaren"
        IMAGE_NAME  = "your-app-name"             // e.g., "project2-app"
        IMAGE_TAG   = "latest"
        DOCKER_CRED_ID = "docker-registry-cred"   // Jenkins credential ID
    }

    options {
        timestamps()
        ansiColor('xterm')
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Dependency Vulnerability Scan (OWASP)') {
            steps {
                sh '''
                    mkdir -p odc-reports
                    docker run --rm \
                        -v "$(pwd)":/src \
                        -v "$(pwd)"/odc-reports:/report \
                        owasp/dependency-check:latest \
                        --scan /src \
                        --format "ALL" \
                        --out /report \
                        --failOnCVSS 7
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'odc-reports/**', fingerprint: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            when {
                anyOf { branch 'main'; branch 'master' }
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CRED_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Check logs and reports.'
        }
        success {
            echo '✅ Build and push successful!'
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
