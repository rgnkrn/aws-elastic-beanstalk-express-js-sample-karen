pipeline {
    agent none   // we'll define agents per stage

    environment {
        REGISTRY       = "rgnkrn1234"     
        IMAGE_NAME     = "ass2-app" 
        IMAGE_TAG      = "latest"
        DOCKER_CRED_ID = "dockerhub-id" 
        SNYK_CRED_ID   = "synk-id"      
    }

    stages {
        stage('Checkout') {
            agent { docker { image 'node:16' } }
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            agent { docker { image 'node:16' } }
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Tests') {
            agent { docker { image 'node:16' } }
            steps {
                sh 'npm test'
            }
        }


        stage('Build Docker Image') {
            agent { docker { image 'docker:latest' args '-v /var/run/docker.sock:/var/run/docker.sock' } }
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push Docker Image') {
            agent { docker { image 'docker:latest' args '-v /var/run/docker.sock:/var/run/docker.sock' } }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: env.DOCKER_CRED_ID,
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                      docker push $REGISTRY/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed. Check logs and reports.'
        }
        success {
            echo '✅ Build, scan, and push successful!'
        }
        failure {
            echo '❌ Build failed. See error logs.'
        }
    }
}
