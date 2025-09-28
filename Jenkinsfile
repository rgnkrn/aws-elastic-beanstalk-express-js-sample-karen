pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            reuseNode true // This helps maintain the workspace context
        }
    }

    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('synk-id')
    }

    stages {
        stage('Checkout') {
            steps {
                // Clones the repository into the workspace
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Installs the Node.js dependencies defined in package.json
                sh 'npm install --save'
            }
        }

        stage('Run Unit Tests') {
            steps {
                // Executes the test suite
                sh 'npm test'
            }
        }

        stage('Dependency Vulnerability Scan') {
            steps {
                // Installs Snyk CLI and runs a security scan
                sh 'npm install -g snyk'
                sh 'snyk auth ${SNYK_TOKEN}'
                // Fails the build if high-severity vulnerabilities are found
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                // A script block is required for variable definitions
                script {
                    // Logs into Docker Hub using credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credential-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        // Defines the full Docker image tag
                        def dockerImage = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        
                        // Builds the Docker image
                        sh "docker build -t ${dockerImage} ."
                        // Pushes the image to the specified registry
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Pipeline finished. Cleaning up the workspace."
                // Cleans the workspace to save disk space
                cleanWs()
            }
        }
    }
}

