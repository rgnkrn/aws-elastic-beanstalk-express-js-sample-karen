pipeline {
    agent {
        docker {
            // Requirement: Use Node 16 as build agent
            image 'node:16'
            args '-u root:root'  // Run as root to avoid permission issues
        }
    }

    environment {
        REGISTRY = "rgnkrn1234"   // 🔹 replace with your Docker Hub username
        IMAGE = "aws-node-app"            // 🔹 name of your app imag
        TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/YOUR_GITHUB_USERNAME/aws-elastic-beanstalk-express-js-sample.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }

        stage('Security Scan') {
            steps {
                // Example using Snyk CLI, must be installed in Jenkins
                sh '''
                    if ! command -v snyk >/dev/null; then
                        npm install -g snyk
                    fi
                    snyk test || exit 1
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t $REGISTRY/$IMAGE:$TAG .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-id', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $REGISTRY/$IMAGE:$TAG
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/npm-debug.log', allowEmptyArchive: true
            junit '**/test-results.xml'
        }
        failure {
            echo "❌ Build failed. Check logs and fix issues."
        }
        success {
            echo "✅ Build, Test, Scan, and Deploy completed successfully."
        }
    }
}
