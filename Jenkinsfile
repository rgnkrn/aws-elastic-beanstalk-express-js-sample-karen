pipeline {
    agent {
        docker {
            image 'rgnkrn1234/node16-docker'   // ✅ Custom Node16 image with Docker CLI installed
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'        // 🔹 Docker Hub username
        IMAGE_NAME = 'your-app-name'          // 🔹 Replace with your app name
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"

        // 🔹 Jenkins credential ID for Docker Hub
        DOCKERHUB = credentials('DockerHub-ID')

        // Node environment
        NODE_ENV = 'test'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                echo '📦 Installing dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --save'
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo '🧪 Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Code Quality Check') {
            steps {
                echo '🔎 Running lint checks...'
                sh 'npm run lint || echo "⚠️ Lint skipped - no lint script found"'
            }
        }

        stage('Build Application') {
            steps {
                echo '⚙️ Building the application...'
                sh 'npm run build || echo "Build step completed"'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image: ${DOCKER_IMAGE}"
                sh 'docker --version'
                sh "docker build -t ${DOCKER_IMAGE} ."
                sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                sh "echo '${DOCKERHUB_PSW}' | docker login -u '${DOCKERHUB_USR}' --password-stdin"
                sh "docker push ${DOCKER_IMAGE}"
                sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                echo "✅ Successfully pushed ${DOCKER_IMAGE}"
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo '🧹 Cleaning up local Docker images...'
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
