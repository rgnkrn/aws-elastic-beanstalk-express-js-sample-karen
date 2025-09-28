pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root'
        }
    }

    environment {
        REGISTRY = "rgnkrn1234"      // your Docker Hub username
        IMAGE = "aws-node-app"
        TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                // Uses the SCM config from Jenkins job
                checkout scm
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

        stage('Build Docker Image') {
            steps {
                sh """
                    docker build -t $REGISTRY/$IMAGE:$TAG .
                """
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
            // junit '**/test-results.xml'  // enable later if XML test reports exist
        }
        failure {
            echo "❌ Build failed. Check logs and fix issues."
        }
        success {
            echo "✅ Build, Test, and Deploy completed successfully."
        }
    }
}
