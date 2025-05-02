import org.csanchez.jenkins.plugins.kubernetes.*

podTemplate(
  label: 'jenkins-k8s-agent',
  containers: [
    containerTemplate(
      name: 'git-ssh',
      image: 'alpine/git',
      command: '/bin/sh',

      ttyEnabled: true,
      volumeMounts: [
        volumeMount(mountPath: '/root/.ssh', name: 'git-ssh-key'),
        volumeMount(mountPath: '/root/.ssh/known_hosts', name: 'ssh-known-hosts', subPath: 'known_hosts')
      ]
    ),
    containerTemplate(
      name: 'kaniko',
      image: 'gcr.io/kaniko-project/executor:debug',
      ttyEnabled: true,
      volumeMounts: [
        volumeMount(mountPath: '/kaniko/.docker', name: 'docker-config')
      ]
    ),
    containerTemplate(
      name: 'kubectl',
      image: 'bitnami/kubectl:latest',
      command: 'cat',
      ttyEnabled: true
    )
  ],
  volumes: [
    secretVolume(secretName: 'git-ssh-key', mountPath: '/root/.ssh'),
    secretVolume(secretName: 'ssh-known-hosts', mountPath: '/root/.ssh', readOnly: true),
    secretVolume(secretName: 'docker-config-secret', mountPath: '/kaniko/.docker', readOnly: true)
  ]
) {
  pipeline {
    agent {
      label 'jenkins-k8s-agent'
    }
    environment {
      IMAGE = "guillemetal/app-hola-mundo:latest"
    }
    stages {
      stage('Clonar repo') {
        steps {
          container('git-ssh') {
            sh '''
              chmod 600 /root/.ssh/id_rsa
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
}
