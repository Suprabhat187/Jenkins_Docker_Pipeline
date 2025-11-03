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
        bat 'npm install'
        bat 'npm test'
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          bat "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} ."
          bat "docker tag ${DOCKER_IMAGE}:${IMAGE_TAG} ${DOCKER_IMAGE}:latest"
        }
      }
    }

    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: env.DOCKERHUB_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
          bat "docker push ${DOCKER_IMAGE}:${IMAGE_TAG}"
          bat "docker push ${DOCKER_IMAGE}:latest"
          bat 'docker logout'
        }
      }
    }

    stage('Deploy Container') {
      steps {
        script {
          bat "docker rm -f jenkins-demo-app || exit 0"
          bat "docker run -d --name jenkins-demo-app -p 4000:3000 ${DOCKER_IMAGE}:${IMAGE_TAG}"
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
