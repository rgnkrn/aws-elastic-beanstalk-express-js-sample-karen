pipeline {
    agent none

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('synk-id')
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
            agent { docker { image 'node:16-alpine' } }
            steps {
                unstash 'source'
                sh 'npm install'
                sh 'npm test'
                sh 'npm install -g snyk'
                sh 'snyk auth ${SNYK_TOKEN}'
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            // Mount Docker socket so this stage can talk to the host daemon
            agent {
                docker {
                    image 'docker:24-cli'   // modern lightweight docker client
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                unstash 'source'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credential-id') {
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
            echo '✅ Build, scan, and push successful!'
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
