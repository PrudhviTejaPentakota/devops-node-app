pipeline {
  agent any

  environment {
    DOCKER_IMAGE = 'prudhviteja6/devops-node-app'
    DOCKER_TAG   = "${env.BUILD_NUMBER}"
    REGISTRY_CREDS = credentials('dockerhub-creds')
  }

  stages {

    stage('Install Dependencies') {
      steps {
        sh 'npm install'
      }
    }

    stage('Run Tests') {
      steps {
        sh 'npm test'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
      }
    }

    stage('Push to Docker Hub') {
      steps {
        sh 'echo $REGISTRY_CREDS_PSW | docker login -u $REGISTRY_CREDS_USR --password-stdin'
        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
        sh "docker push ${DOCKER_IMAGE}:latest"
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sshagent(['ec2-ssh-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@16.112.219.63 \
            "kubectl set image deployment/nodeapp \
             nodeapp=prudhviteja6/devops-node-app:${BUILD_NUMBER}"
          '''
        }
      }
    }

  }

  post {
    success { echo 'Pipeline succeeded! App deployed.' }
    failure { echo 'Pipeline FAILED. Check the logs above.' }
    always  { sh 'docker logout' }
  }
}
