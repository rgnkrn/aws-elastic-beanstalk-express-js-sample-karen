pipeline {
    agent {
        docker {
            image 'node:16'
            args '''
              -u root:root
              --priviledged
              -v /usr/bin/docker:/usr/bin/docker
              -v /certs/client:/certs/client:ro
            '''
        }
    }

    environment {
        REGISTRY       = "rgnkrn1234"     // your DockerHub username
        IMAGE_NAME     = "ass2-app" // change to your repo name
        IMAGE_TAG      = "latest"
        DOCKER_CRED_ID = "dockerhub-id" // Jenkins DockerHub credentials
        SNYK_CRED_ID   = "snyk-id"          // Jenkins Snyk API token

        DOCKER_HOST     = "tcp://docker:2376"
        DOCKER_CERT_PATH = "/certs/client"
        DOCKER_TLS_VERIFY = "1"
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Check Docker Connectivity') {
            steps {
                sh 'docker version'
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

        stage('Dependency Vulnerability Scan (Snyk)') {
            steps {
                withCredentials([string(credentialsId: env.SNYK_CRED_ID, variable: 'SNYK_TOKEN')]) {
                    sh '''
                      docker run --rm \
                        -e SNYK_TOKEN=$SNYK_TOKEN \
                        -v "$(pwd)":/app \
                        -w /app \
                        snyk/snyk test --severity-threshold=high
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
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
            echo '✅ Build, scan, and push successful!'
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
