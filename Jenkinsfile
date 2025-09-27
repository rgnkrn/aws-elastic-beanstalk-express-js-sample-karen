pipeline {
  agent {
    // Requirement: Use Node 16 Docker image as the build agent
    docker {
      image 'node:16'
      args '-u root:root'   // allow installing tools if needed
    }
  }


  environment {
    // ==== CHANGE THESE FOR YOUR ENV ====
    // Use Docker Hub (simplest) or GHCR (see notes below)
    REGISTRY   = 'rgnkrn1234'   // e.g., 'erginakaren'
    IMAGE_NAME = 'assignment2-app'             // e.g., 'project2-app'
    IMAGE_TAG  = 'latest'
    // Jenkins credentials (Manage Jenkins -> Credentials)
    // - For Docker Hub: create "Username with password" id=docker-registry-cred
    DOCKER_CRED_ID = '0038f5ff-994e-44a6-b23b-b7e5d9dde26d'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install Dependencies') {
      steps {
        // Requirement: use npm install --save
        sh 'npm install --save'
      }
    }

    stage('Unit Tests') {
      steps {
        // If your project emits JUnit XML, add: -- --reporters=junit ...
        sh 'npm test'
      }
      post {
        always {
          // If you have JUnit XML results (e.g., in reports/junit.xml), publish them:
          // junit 'reports/**/*.xml'
        }
      }
    }

    stage('Dependency Vulnerability Scan (OWASP)') {
      steps {
        sh '''
          mkdir -p odc-reports
          docker run --rm \
            -v "$(pwd)":/src \
            -v "$(pwd)"/odc-reports:/report \
            owasp/dependency-check:latest \
            --scan /src \
            --format "ALL" \
            --out /report \
            --failOnCVSS 7
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: 'odc-reports/**', fingerprint: true
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
      }
    }

    stage('Push Docker Image') {
      when {
        anyOf {
          branch 'main'; branch 'master'   // push only on mainline
        }
      }
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
    success {
      echo '✅ Pipeline completed successfully.'
    }
    failure {
      echo '❌ Pipeline failed. Check console and odc-reports for details.'
    }
    always {
      // Keep a copy of npm logs if useful
      // archiveArtifacts artifacts: 'npm-debug.log', allowEmptyArchive: true
    }
  }
}
