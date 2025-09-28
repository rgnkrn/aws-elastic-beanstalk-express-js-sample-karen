pipeline {
    // Defines the agent for the entire pipeline.
    // All stages will run inside a container based on this image.
    agent {
        docker {
            image 'node:16-alpine'
        }
    }

    // Environment variables available to all stages.
    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
    }

    stages {
        stage('Checkout') {
            steps {
                // Clones the source code from the repository, which is needed for the build.
                checkout scm
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                // A 'script' block is needed for defining variables and logic.
                script {
                    // This block securely handles Docker registry authentication and image operations
                    // using Jenkins's built-in Docker pipeline support.
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credential-id') {
                        
                        // Creates a unique image tag using the registry, app name, and build number.
                        def dockerImageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        
                        // Builds the Docker image from the Dockerfile in the repo.
                        def customImage = docker.build(dockerImageTag)

                        // Pushes the built image to your Docker Hub registry.
                        customImage.push()
                    }
                }
            }
        }
    }
}

