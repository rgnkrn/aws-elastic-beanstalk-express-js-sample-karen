pipeline {
    agent {
    docker {
        image 'node:16'
        args '-u root:root'
        reuseNode true
    }
}

    environment {
        REGISTRY    = "rgnkrn1234"        // your Docker Hub username
        IMAGE_NAME  = "express-sample"    // change to your app name
        IMAGE_TAG   = "latest"
        DOCKER_CRED_ID = "d0038f5ff-994e-44a6-b23b-b7e5d9dde26d"  // use a readable ID in Jenkins
        DOCKER_HOST = "tcp://docker:2376"
        DOCKER_TLS_VERIFY = "1"
        DOCKER_CERT_PATH = "/certs/client"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Node') {
            steps {
                sh 'node -v && npm -v'
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
            echo 'âœ… Build and push successful!'
        }
        failure {
            echo 'âŒ Build failed. See error logs.'
        }
    }
}
