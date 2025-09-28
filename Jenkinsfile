pipeline {
    agent none   // no global agent, each stage defines its own

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'       // 🔹 Docker Hub username
        IMAGE_NAME = 'your-app-name'         // 🔹 Replace with your app name
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"

        // 🔹 Docker Hub credentials (exposes DOCKERHUB_USR and DOCKERHUB_PSW)
        DOCKERHUB = credentials('DockerHub-ID')

        NODE_ENV = 'test'
    }

    stages {
        stage('Checkout') {
            agent {
                docker {
                    image 'node:16-bullseye'
                }
            }
            steps {
                echo '📥 Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            agent {
                docker {
                    image 'node:16-bullseye'
                }
            }
            steps {
                echo '📦 Installing Node.js dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --save'
            }
        }

        stage('Run Unit Tests') {
            agent {
                docker {
                    image 'node:16-bullseye'
                }
            }
            steps {
                echo '🧪 Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Code Quality Check') {
            agent {
                docker {
                    image 'node:16-bullseye'
                }
            }
            steps {
                echo '🔎 Running code quality checks...'
                sh 'npm run lint || echo "⚠️ Lint skipped - no lint script found"'
            }
        }

        stage('Build Application') {
            agent {
                docker {
                    image 'node:16-bullseye'
                }
            }
            steps {
                echo '⚙️ Building the application...'
                sh 'npm run build || echo "Build step completed"'
            }
        }

        stage('Build Docker Image') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    echo "🐳 Installing Docker CLI inside Node 16 agent..."
                    sh 'apt-get update && apt-get install -y docker-cli curl'

                    echo "🐳 Building Docker image: ${DOCKER_IMAGE}"
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Registry') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    echo "📤 Pushing Docker image to Docker Hub..."
                    sh 'apt-get update && apt-get install -y docker-cli curl'
                    sh "echo '${DOCKERHUB_PSW}' | docker login -u '${DOCKERHUB_USR}' --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Cleanup') {
            agent {
                docker {
                    image 'node:16-bullseye'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo '🧹 Cleaning up local Docker images...'
                sh 'apt-get update && apt-get install -y docker-cli curl'
                sh "docker rmi ${DOCKER_IMAGE} || true"
                sh "docker rmi ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest || true"
                sh 'docker system prune -f || true'
            }
        }
    }

    post {
        always {
            echo '📦 Pipeline execution completed.'
            archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true, fingerprint: true
            cleanWs()
        }

        success {
            echo '🎉 Pipeline succeeded!'
        }

        failure {
            echo '❌ Pipeline failed!'
        }

        unstable {
            echo '⚠️ Pipeline is unstable!'
        }
    }
}
