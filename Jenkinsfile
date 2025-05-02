podTemplate(
  label: 'jenkins-k8s-agent',
  containers: [
    containerTemplate(
      name: 'git-ssh',
      image: 'alpine/git',
      command: 'sleep',
      args: 'infinity',
      tty: true,
      volumeMounts: [
        // Montar el Secret en una ruta alternativa
        [ mountPath: '/tmp/.ssh', name: 'git-ssh-key' ],
        [ mountPath: '/tmp/known_hosts', name: 'ssh-known-hosts', subPath: 'known_hosts' ]
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
      command: 'sleep',
      args: 'infinity',
      volumeMounts: [
        [ mountPath: '/root/.kube', name: 'kube-config-secret' ]
      ]
    )
  ],
  volumes: [
    secretVolume(secretName: 'git-ssh-key', mountPath: '/tmp/.ssh'),
    secretVolume(secretName: 'ssh-known-hosts', mountPath: '/tmp/known_hosts'),
    secretVolume(secretName: 'docker-config-secret', mountPath: '/kaniko/.docker'),
    secretVolume(secretName: 'kube-config-secret', mountPath: '/root/.kube')
  ]
) {
  node('jenkins-k8s-agent') {
    stage('Clonar repo') {
      container('git-ssh') {
        sh '''
          // Copiar las claves a /root/.ssh y asignar permisos
          mkdir -p /root/.ssh
          cp /tmp/.ssh/id_rsa /root/.ssh/id_rsa
          cp /tmp/known_hosts /root/.ssh/known_hosts
          chmod 600 /root/.ssh/id_rsa
        '''
        git url: 'git@github.com:bonanza1958/app-hola-mundo.git'
      }
    }
    // ... (etapas posteriores)
  }
}