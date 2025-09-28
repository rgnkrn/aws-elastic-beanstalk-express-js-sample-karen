pipeline {
    agent none

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        IMAGE_TAG       = 'latest'
        SNYK_TOKEN      = credentials('snyk-id')   // replace with your Snyk creds ID
        DOCKER_CREDS    = 'dockerhub-credential-id' // replace with your DockerHub creds ID
    }

    stages {
        stage('Checkout') {
            agent { docker { image 'alpine/git' } }
            steps {
                checkout scm
                stash name: 'source', includes: '**/*'
            }
        }

        stage('Install, Test & Scan') {
            agent { docker { image 'node:16' } }
            steps {
                unstash 'source'
                sh 'npm install --save'
                sh 'npm test'
                sh 'npm install -g snyk'
                sh 'snyk auth ${SNYK_TOKEN}'
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            agent { docker { image 'node:16' } }
            steps {
                unstash 'source'
                // Install Docker CLI inside node:16 container
                sh '''
                  apt-get update
                  apt-get install -y docker.io
                '''
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDS) {
                        def dockerImageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        sh "docker build -t ${dockerImageTag} ."
                        sh "docker push ${dockerImageTag}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Check logs and reports.'
        }
        success {
            echo '✅ Build, test, scan, and push successful!'
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
