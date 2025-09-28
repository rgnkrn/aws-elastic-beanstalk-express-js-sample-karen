pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            reuseNode true
        }
    }

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('synk-id')
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

        stage('Run Unit Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Dependency Vulnerability Scan') {
            steps {
                sh 'npm install -g snyk'
                sh 'snyk auth ${SNYK_TOKEN}'
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credential-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        def dockerImage = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        
                        sh "docker build -t ${dockerImage} ."
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                // The docker logout command needs to be inside the agent context to succeed
                // but we can't guarantee that context in the post block.
                // It's safer to let the credentials expire.
                echo "Pipeline finished. Cleaning up workspace."
                cleanWs()
            }
        }
    }
}

