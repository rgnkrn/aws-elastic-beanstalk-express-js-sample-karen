// Declarative Pipeline
pipeline {
    // 1.a. Use Node 16 Docker image as the build agent.
    agent {
        docker {
            image 'node:16-alpine'
            // Expose the Docker socket to build/push images from within the container
            args '-v /var/run/docker.sock:/var/run/docker.sock'
            // Ensures all stages run on the same container, preserving the workspace context.
            // This helps prevent "could not be found" and "MissingContextVariableException" errors.
            reuseNode true
        }
    }

  // Environment variables used throughout the pipeline
    environment {
        // Define your Docker registry URL (e.g., 'your-dockerhub-username')
        DOCKER_REGISTRY = 'rgnkrn1234'
        // Define the application/image name
        APP_NAME        = 'my-ass2-app'
        // Use Jenkins credentials for the Snyk token
        SNYK_TOKEN      = credentials('synk-id')
    }

    stages {
        // Stage 1: Checkout source code from the repository
        stage('Checkout') {
            steps {
                script {
                    echo "Checking out source code..."
                    checkout scm
                }
            }
        }

        // Stage 2: Install project dependencies
        stage('Install Dependencies') {
            steps {
                script {
                    echo "Installing NPM dependencies..."
                    // 1.b.ii. Install dependencies
                    sh 'npm install'
                }
            }
        }

        // Stage 3: Run Unit Tests
        stage('Run Unit Tests') {
            steps {
                script {
                    echo "Running unit tests..."
                    // 1.b.ii. Run unit tests
                    sh 'npm test'
                }
            }
        }

        // Stage 4: Security Scan with Snyk
        stage('Dependency Vulnerability Scan') {
            steps {
                script {
                    echo "Running Snyk vulnerability scan..."
                    // Install Snyk CLI globally in the container
                    sh 'npm install -g snyk'
                    // Authenticate with the Snyk token from Jenkins credentials
                    sh 'snyk auth ${SNYK_TOKEN}'

                    // 2.a. Integrate a dependency vulnerability scanner
                    // 2.b. Fail the pipeline if High/Critical issues are found
                    echo "Scanning for High or Critical severity vulnerabilities..."
                    sh 'snyk test --severity-threshold=high'
                }
            }
        }

        // Stage 5: Build and Push Docker Image
        stage('Build and Push Docker Image') {
            steps {
                script {
                    echo "Building and pushing Docker image..."
                    // Use Docker Hub credentials stored in Jenkins
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credential-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Log in to the Docker registry
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"

                        // Define the full image tag (e.g., your-docker-registry/my-node-app:1)
                        def dockerImage = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"

                        // 1.b.ii. Build the Docker image
                        echo "Building image: ${dockerImage}"
                        sh "docker build -t ${dockerImage} ."

                        // 1.b.ii. Push the Docker image to the registry
                        echo "Pushing image: ${dockerImage}"
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
    }

    // Post-build actions that run regardless of the pipeline's status
    post {
        always {
            script {
                echo "Pipeline finished. Cleaning up workspace..."
                // Log out from Docker to clean up credentials
                sh 'docker logout'
                cleanWs()
            }
        }
    }
}

