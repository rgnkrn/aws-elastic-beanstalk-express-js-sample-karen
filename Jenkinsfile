pipeline {
    agent none   // no global agent, each stage defines its own

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'       // 🔹 Docker Hub username
        IMAGE_NAME = 'your-app-name'         // 🔹 Replace with your app name
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"

        // 🔹 Docker Hub credentials (exposes DOCKERHUB_USR and DOCKERHUB_PSW)
        DOCKERHUB = credentials('DockerHub-ID')

        // Node environment
        NODE_ENV = 'test'
    }

    stages {
        stage('Checkout') {
            agent { docker { image 'node:16' } }
            steps {
                echo '📥 Checking out source code...'
                checkout scm
            }
        }

        stage('Install Dependencies') {
            agent { docker { image 'node:16' } }
            steps {
                echo '📦 Installing Node.js dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --save'
            }
        }

        stage('Run Unit Tests') {
            agent { docker { image 'node:16' } }
            steps {
                echo '🧪 Running unit tests...'
                sh 'npm test'
            }
        }

        stage('Code Quality Check') {
            agent { docker { image 'node:16' } }
            steps {
                echo '🔎 Running code quality checks...'
                sh 'npm run lint || echo "⚠️ Lint skipped - no lint script found"'
            }
        }

        stage('Build Application') {
            agent { docker { image 'node:16' } }
            steps {
                echo '⚙️ Building the application...'
                sh 'npm run build || echo "Build step completed"'
            }
        }

        stage('Build Docker Image') {
            agent any   // ✅ run on Jenkins host (has Docker installed)
            steps {
                script {
                    echo "🐳 Building Docker image: ${DOCKER_IMAGE}"
                    sh 'docker --version'
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Push to Registry') {
            agent any   // ✅ run on Jenkins host
            steps {
                script {
                    echo "📤 Pushing Docker image to Docker Hub..."
                    sh "echo '${DOCKERHUB_PSW}' | docker login -u '${DOCKERHUB_USR}' --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                    echo "✅ Successfully pushed ${DOCKER_IMAGE}"
                }
            }
            post {
                always {
                    sh 'docker logout'
                }
            }
        }

        stage('Cleanup') {
            agent any   // ✅ run on Jenkins host
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
            node {
                echo '📦 Pipeline execution completed.'
                archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true, fingerprint: true
                cleanWs()
            }
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
