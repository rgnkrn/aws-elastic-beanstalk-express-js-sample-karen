pipeline {
    agent none   // each stage defines its own agent

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'         // your DockerHub username/namespace
        APP_NAME        = 'my-ass2-app'        // image name
        IMAGE_TAG       = "build-${BUILD_NUMBER}"
        SNYK_TOKEN      = credentials('snyk-id')       // Jenkins secret text credential
        DOCKER_CREDS    = 'dockerhub-credential-id'    // Jenkins username/password credential
    }

    stages {
        stage('Checkout') {
            agent { docker { image 'node:16' } }
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
            agent {
                docker {
                    image 'node:16'
                    args '--network jenkins -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                unstash 'source'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDS) {
                        def dockerImageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
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
            echo "✅ Build, scan, and push successful! Image: ${DOCKER_REGISTRY}/${APP_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
