pipeline {
    // Defines the agent for the entire pipeline.
    // All stages will run inside a container based on this image.
    agent {
        docker {
            image 'node:16-alpine'
        }
    }

    // Environment variables available to all stages.
    // 'credentials()' retrieves secrets from the Jenkins credential store.
    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        // IMPORTANT: 'synk-id' must be a 'Secret text' credential in Jenkins.
        SNYK_TOKEN      = credentials('synk-id')
    }

    stages {
        stage('Checkout') {
            steps {
                // Clones the source code from the repository.
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                // Installs Node.js dependencies from package.json.
                sh 'npm install --save'
            }
        }

        stage('Run Unit Tests') {
            steps {
                // Executes the project's unit tests.
                sh 'npm test'
            }
        }

        stage('Dependency Vulnerability Scan') {
            steps {
                // Installs the Snyk CLI globally in the container.
                sh 'npm install -g snyk'
                // Authenticates Snyk with the provided token.
                sh 'snyk auth ${SNYK_TOKEN}'
                // Scans for vulnerabilities. The build will fail if any 'high'
                // severity issues are found, as required by the assignment.
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                // A 'script' block is needed for defining variables and logic.
                script {
                    // This block securely accesses Docker Hub credentials.
                    // IMPORTANT: 'dockerhub-credential-id' must be a 'Username with password' credential.
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credential-id', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        // Logs into the Docker registry.
                        sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                        
                        // Creates a unique image tag using the registry, app name, and build number.
                        def dockerImage = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        
                        // Builds the Docker image from the Dockerfile in the repo.
                        sh "docker build -t ${dockerImage} ."
                        // Pushes the built image to your Docker Hub registry.
                        sh "docker push ${dockerImage}"
                    }
                }
            }
        }
    }
}

