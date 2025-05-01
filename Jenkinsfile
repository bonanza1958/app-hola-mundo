pipeline {
  agent any

  environment {
    IMAGE = "guillemetal/app-hola-mundo:latest"
    REGISTRY = "docker.io"
  }

  stages {
    stage('Clonar repo') {
      steps {
        git 'git@github.com:bonanza1958/app-hola-mundo.git'
      }
    }

    stage('Build imagen Docker') {
      steps {
        sh 'docker build -t $IMAGE .'
      }
    }

    stage('Push a DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
          sh 'docker push $IMAGE'
        }
      }
    }

    stage('Desplegar en Kubernetes') {
      steps {
        sh 'kubectl set image deployment/hola-mundo nginx=$IMAGE -n default'
      }
    }
  }
}
