pipeline {
    // Use a primary agent that has the Docker client and Git installed.
    // This agent will orchestrate all other actions.
    agent {
        docker { image 'docker:git' }
    }

    // Environment variables available to all stages.
    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('synk-id')
    }

    stages {
        stage('Checkout') {
            steps {
                // This works because the 'docker:git' agent has Git.
                checkout scm
            }
        }

        // We combine the Node.js steps into a single stage for efficiency.
        stage('Install, Test & Scan') {
            steps {
                // The '.inside()' block runs these commands in a temporary container
                // based on the 'node:16-alpine' image, which has the npm tools.
                docker.image('node:16-alpine').inside {
                    sh 'npm install --save'
                    sh 'npm test'
                    sh 'npm install -g snyk'
                    sh 'snyk auth ${SNYK_TOKEN}'
                    sh 'snyk test --severity-threshold=high'
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // This now works because the primary 'docker:git' agent has the
                    // Docker client and can communicate with your DinD service.
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credential-id') {
                        def dockerImageTag = "${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}"
                        def customImage = docker.build(dockerImageTag)
                        customImage.push()
                    }
                }
            }
        }
    }
}

