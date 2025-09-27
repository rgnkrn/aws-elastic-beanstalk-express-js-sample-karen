pipeline {
    agent {
        docker {
            image 'node:16'
            args '-u root:root'
        }
    }

    environment {
        DOCKER_HOST = "tcp://172.20.0.3:2376"
        DOCKER_TLS_VERIFY = "1"
        DOCKER_CERT_PATH = "/certs/client"
    }

    stages {
        stage('Check Docker TLS') {
            steps {
                sh 'docker version'
            }
        }
        stage('Check Node/NPM') {
            steps {
                sh 'node -v && npm -v'
            }
        }
    }
}
