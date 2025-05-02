podTemplate(
  label: 'jenkins-k8s-agent',
  containers: [
    containerTemplate(
      name: 'git-ssh',
      image: 'alpine/git',
      command: '/bin/sh',
      ttyEnabled: true,
      volumeMounts: [
        [ mountPath: '/root/.ssh', name: 'git-ssh-key' ],
        [ mountPath: '/root/.ssh/known_hosts', name: 'ssh-known-hosts', subPath: 'known_hosts' ]
      ]
    ),
    containerTemplate(
      name: 'kaniko',
      image: 'gcr.io/kaniko-project/executor:debug',
      command: 'sleep',
      args: 'infinity',
      volumeMounts: [
        [ mountPath: '/kaniko/.docker', name: 'docker-config-secret' ]
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
  node('jenkins-k8s-agent') {
    stage('Clonar repo') {
      container('git-ssh') {
        sh 'chmod 600 /root/.ssh/id_rsa'
        git credentialsId: 'git-ssh', url: 'git@github.com:bonanza1958/app-hola-mundo.git'
      }
    }

    stage('Verificar Docker config') {
      container('kaniko') {
        sh 'ls -l /kaniko/.docker'
        sh 'cat /kaniko/.docker/config.json || echo "Archivo no encontrado"'
      }
    }

    stage('Build y Push con Kaniko') {
      container('kaniko') {
        withEnv(["DOCKER_CONFIG=/kaniko/.docker"]) {
          sh '''
            /kaniko/executor \
              --context `pwd` \
              --dockerfile `pwd`/Dockerfile \
              --destination=guillemetal/app-hola-mundo:latest \
              --verbosity=info
          '''
        }
      }
    }

    stage('Desplegar en Kubernetes') {
      container('kubectl') {
        sh 'kubectl set image deployment/hola-mundo nginx=guillemetal/app-hola-mundo:latest -n default'
      }
    }
  }
}
