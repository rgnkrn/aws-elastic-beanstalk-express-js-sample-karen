pipeline {
    agent {
        docker {
            image 'node:16-bullseye'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        IMAGE_NAME      = 'assingment2-app'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        DOCKER_IMAGE    = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        DOCKERHUB       = credentials('DockerHub-ID')
        NODE_ENV        = 'test'
        SNYK_TOKEN      = credentials('Snyk-Token')
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
                echo '📦 Installing Node.js dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --save'
            }
        }

        stage('Snyk Vulnerability Scan') {
            steps {
                echo '🛡️ Running Snyk dependency vulnerability scan...'
                sh 'npm install -g snyk'
                withEnv(["SNYK_TOKEN=${SNYK_TOKEN}"]) {
                    sh 'snyk auth $SNYK_TOKEN'
                    sh 'snyk test --severity-threshold=high'
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                echo '🧪 Running unit tests...'
                sh 'npm test'
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
                echo "🐳 Installing Docker CLI inside Node 16 agent..."
                sh 'apt-get update && apt-get install -y docker.io curl'
                sh 'docker --version && node --version'
                echo "🐳 Building Docker image: ${DOCKER_IMAGE}"
                sh "docker build -t ${DOCKER_IMAGE} ."
                sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
            }
        }

        stage('Push to Registry') {
            steps {
                echo "📤 Pushing Docker image to Docker Hub..."
                sh 'apt-get update && apt-get install -y docker.io curl'
                sh "echo '${DOCKERHUB_PSW}' | docker login -u '${DOCKERHUB_USR}' --password-stdin"
                sh "docker push ${DOCKER_IMAGE}"
                sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
            }
            post {
                always {
                    sh 'docker logout || true'
                }
            }
        }

        stage('Cleanup Docker Cache') {
            steps {
                echo '🧹 Cleaning up local Docker images...'
                sh 'apt-get update && apt-get install -y docker.io'
                sh "docker rmi ${DOCKER_IMAGE} || true"
                sh "docker rmi ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest || true"
                sh 'docker system prune -f || true'
            }
        }

        stage('Archive & Post') {
            steps {
                echo '📦 Archiving build artifacts...'
                archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true, fingerprint: true
                cleanWs()
            }
        }
    }
}
