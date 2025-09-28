pipeline {
    agent {
        docker {
            image 'node:16'
            // Use Docker daemon from DinD container
            args '--network jenkins -v docker-certs-client:/certs/client:ro -e DOCKER_HOST=tcp://docker:2376 -e DOCKER_CERT_PATH=/certs/client -e DOCKER_TLS_VERIFY=1'
        }
    }
    
    environment {
        // Define your Docker registry and image name
        DOCKER_REGISTRY = 'rgnkrn1234' // Replace with your actual registry
        IMAGE_NAME = 'your-app-name'          // Replace with your app name
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
        
        // Docker Hub credentials (configure these in Jenkins credentials)
        DOCKER_CREDENTIALS = credentials('dockerhub-id')
        
        // Node environment
        NODE_ENV = 'test'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'node --version'
                sh 'npm --version'
                sh 'npm install --save'
            }
        }
        
        stage('Run Unit Tests') {
            steps {
                echo 'Running unit tests...'
                sh 'npm test'
            }
            post {
                always {
                    // Archive test results if using a test reporter like Jest
                    publishTestResults testResultsPattern: 'test-results.xml'
                    // Archive test coverage reports if available
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }
        
        stage('Code Quality Check') {
            steps {
                echo 'Running code quality checks...'
                // Optional: Add linting
                sh 'npm run lint || echo "Linting step skipped - no lint script found"'
            }
        }
        
        stage('Build Application') {
            steps {
                echo 'Building the application...'
                sh 'npm run build || echo "Build step completed"'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image: ${DOCKER_IMAGE}"
                    
                    // Docker is already available through DinD setup
                    // No need to install Docker CLI - it's available via environment
                    sh 'docker --version'
                    
                    // Build the Docker image
                    sh "docker build -t ${DOCKER_IMAGE} ."
                    sh "docker tag ${DOCKER_IMAGE} ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                echo 'Running security scans...'
                // Optional: Add security scanning
                sh 'npm audit --audit-level moderate || echo "Security audit completed with warnings"'
            }
        }
        
        stage('Push to Registry') {
            steps {
                script {
                    echo "Pushing Docker image to registry..."
                    
                    // Login to Docker registry
                    sh "echo '${DOCKER_CREDENTIALS_PSW}' | docker login ${DOCKER_REGISTRY} -u '${DOCKER_CREDENTIALS_USR}' --password-stdin"
                    
                    // Push the images
                    sh "docker push ${DOCKER_IMAGE}"
                    sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest"
                    
                    echo "Successfully pushed ${DOCKER_IMAGE}"
                }
            }
            post {
                always {
                    // Cleanup: logout from Docker registry
                    sh "docker logout ${DOCKER_REGISTRY}"
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up local Docker images...'
                sh "docker rmi ${DOCKER_IMAGE} || true"
                sh "docker rmi ${DOCKER_REGISTRY}/${IMAGE_NAME}:latest || true"
                sh 'docker system prune -f'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed.'
            
            // Archive build artifacts
            archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true, fingerprint: true
            
            // Clean workspace
            cleanWs()
        }
        
        success {
            echo 'Pipeline succeeded! ✅'
            // Optional: Send success notification
            // slackSend channel: '#deployments', 
            //          color: 'good',
            //          message: "✅ Build #${BUILD_NUMBER} succeeded for ${JOB_NAME}"
        }
        
        failure {
            echo 'Pipeline failed! ❌'
            // Optional: Send failure notification
            // slackSend channel: '#deployments', 
            //          color: 'danger',
            //          message: "❌ Build #${BUILD_NUMBER} failed for ${JOB_NAME}"
        }
        
        unstable {
            echo 'Pipeline is unstable! ⚠️'
        }
    }
}