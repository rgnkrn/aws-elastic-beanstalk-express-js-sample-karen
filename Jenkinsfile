pipeline {
    agent none

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'   // your DockerHub username/registry
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('snyk-id')
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
            agent { docker { image 'docker:20.10' args '--network jenkins -v /var/run/docker.sock:/var/run/docker.sock' } }
            steps {
                unstash 'source'
                script {
                    def tag = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                    sh "docker build -t ${tag} ."
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh """
                          echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                          docker push ${tag}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build, scan and push completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed, check logs.'
        }
    }
}
