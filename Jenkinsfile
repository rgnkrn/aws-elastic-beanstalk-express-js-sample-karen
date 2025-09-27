pipeline {
  agent {
    // Requirement: Use Node 16 Docker image as the build agent
    docker {
      image 'node:16'
      args '-u root:root'   // allow installing tools if needed
    }
  }

    environment {
        REGISTRY    = "rgnkrn1234"   // e.g., "erginakaren"
        IMAGE_NAME  = "your-app-name"             // e.g., "project2-app"
        IMAGE_TAG   = "latest"
        DOCKER_CRED_ID = "0038f5ff-994e-44a6-b23b-b7e5d9dde26d"   // Jenkins credential ID
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
