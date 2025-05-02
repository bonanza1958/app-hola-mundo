pipeline {
  agent {
    kubernetes {
      label 'jenkins-k8s-agent'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/label: jenkins-k8s-agent
spec:
  containers:
  - name: git-ssh
    image: alpine/git
    command:
    - cat
    tty: true
    volumeMounts:
    - name: git-ssh-key
      mountPath: /root/.ssh
  - name: kaniko
    image: gcr.io/kaniko-project/executor:latest
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-config
      mountPath: /kaniko/.docker
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  volumes:
  - name: git-ssh-key
    secret:
      secretName: git-ssh-key
  - name: ssh-known-hosts
    configMap:
      name: ssh-known-hosts
      items:
      - key: known_hosts
        path: known_hosts
  - name: docker-config
    secret:
      secretName: docker-config-secret
"""
    }
  }

  environment {
    IMAGE = "docker.io/guillemetal/app-hola-mundo:latest"
  }

  stages {
    stage('Clonar repo') {
      steps {
        container('git-ssh') {
          sh '''
            chmod 600 /root/.ssh/id_rsa
            ssh-keyscan github.com >> /root/.ssh/known_hosts
          '''
          git credentialsId: 'git-ssh', url: 'git@github.com:bonanza1958/app-hola-mundo.git'
        }
      }
    }

    stage('Build y Push con Kaniko') {
      steps {
        container('kaniko') {
          withEnv(["DOCKER_CONFIG=/kaniko/.docker"]) {
            sh '''
              /kaniko/executor \
                --context `pwd` \
                --dockerfile `pwd`/Dockerfile \
                --destination=$IMAGE \
                --verbosity=info
            '''
          }
        }
      }
    }

    stage('Desplegar en Kubernetes') {
      steps {
        container('kubectl') {
          sh 'kubectl set image deployment/hola-mundo nginx=$IMAGE -n default'
        }
      }
    }
  }
}
