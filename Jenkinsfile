pipeline {
    // No top-level agent is defined. Each stage will specify its own agent
    // to ensure commands are run in the correct environment.
    agent none

    // Environment variables available to all stages.
    environment {
        DOCKER_REGISTRY = 'rgnkrn1234'
        APP_NAME        = 'my-ass2-app'
        SNYK_TOKEN      = credentials('synk-id')
    }

    stages {
        stage('Checkout') {
            // This stage uses an agent with Git pre-installed.
            agent { docker { image 'docker:git' } }
            steps {
                checkout scm
                // 'stash' saves the workspace files (the source code) so they
                // can be loaded into the next stage's different agent.
                stash name: 'source', includes: '**/*'
            }
        }

        stage('Install, Test & Scan') {
            // This stage uses a Node.js agent for all npm-related tasks.
            agent { docker { image 'node:16-alpine' } }
            steps {
                // 'unstash' restores the files saved from the previous stage.
                unstash 'source'
                sh 'npm install --save'
                sh 'npm test'
                sh 'npm install -g snyk'
                sh 'snyk auth ${SNYK_TOKEN}'
                sh 'snyk test --severity-threshold=high'
            }
        }

        stage('Build and Push Docker Image') {
            // This stage returns to an agent with the Docker client.
            agent { docker { image 'docker:git' } }
            steps {
                // The source code is unstashed again to provide the build context
                // (including the Dockerfile) for the docker.build command.
                unstash 'source'
                script {
                    // This will now execute correctly because the 'docker:git' agent
                    // has the Docker client and can communicate with the DinD service.
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

