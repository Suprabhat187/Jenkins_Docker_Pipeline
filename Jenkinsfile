pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = 'dockerhub-creds'  // Jenkins credentials id
    DOCKER_IMAGE = "yourdockerhubusername/jenkins-demo-app"
    IMAGE_TAG = "${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Install & Test') {
      steps {
        sh 'npm install'
        sh 'npm test'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
          sh "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
          sh "docker push ${DOCKER_IMAGE}:latest"
          sh 'docker logout'
        }
      }
    }

    stage('Deploy Container') {
      steps {
        script {
          sh "docker rm -f jenkins-demo-app || true"
          sh "docker run -d --name jenkins-demo-app -p 4000:3000 ${DOCKER_IMAGE}:${IMAGE_TAG}"
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: '**/package.json', fingerprint: true
    }
  }
}
