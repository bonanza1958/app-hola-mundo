podTemplate(
  label: 'jenkins-docker-agent',
  containers: [
    containerTemplate(
      name: 'docker',
      image: 'docker:24.0.7-dind',  // Docker-in-Docker (DinD)
      command: 'dockerd-entrypoint.sh',
      args: '--host tcp://0.0.0.0:2375',
      privileged: true,  // Necesario para DinD
      ttyEnabled: true,
      resourceRequestCpu: '500m',
      resourceLimitCpu: '1000m',
      resourceRequestMemory: '512Mi',
      resourceLimitMemory: '1Gi',
      volumeMounts: [
        [mountPath: '/root/.docker', name: 'docker-config-secret'],  // Credenciales
        [mountPath: '/var/run/docker.sock', name: 'docker-sock']  // Socket de Docker
      ]
    ),
    containerTemplate(
      name: 'git-ssh',
      image: 'alpine/git',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      volumeMounts: [
        [mountPath: '/tmp/.ssh', name: 'git-ssh-key'],
        [mountPath: '/tmp/known_hosts', name: 'ssh-known-hosts', subPath: 'known_hosts']
      ]
    ),
    containerTemplate(
      name: 'kubectl',
      image: 'bitnami/kubectl:latest',
      command: 'sleep',
      args: 'infinity',
      volumeMounts: [
        [mountPath: '/root/.kube', name: 'kube-config-secret']
      ]
    )
  ],
  volumes: [
    secretVolume(secretName: 'docker-config-secret', mountPath: '/root/.docker'),
    hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock'),  // Montar socket del host
    secretVolume(secretName: 'git-ssh-key', mountPath: '/tmp/.ssh'),
    secretVolume(secretName: 'ssh-known-hosts', mountPath: '/tmp/known_hosts'),
    secretVolume(secretName: 'kube-config-secret', mountPath: '/root/.kube')
  ]
) {
  node('jenkins-docker-agent') {
    stage('Clonar repo') {
      container('git-ssh') {
        sh '''
          mkdir -p /root/.ssh
          cp /tmp/.ssh/id_rsa /root/.ssh/id_rsa
          cp /tmp/known_hosts /root/.ssh/known_hosts
          chmod 600 /root/.ssh/id_rsa
        '''
        git url: 'git@github.com:bonanza1958/app-hola-mundo.git'
      }
    }

    stage('Build y Push con Docker') {
      container('docker') {
        withEnv(["DOCKER_HOST=tcp://localhost:2375"]) {
          sh '''
            // Autenticarse en Docker Hub
            docker login -u $DOCKER_USER -p $DOCKER_PASSWORD

            // Construir la imagen
            docker build -t guillemetal/app-hola-mundo:latest .

            // Subir la imagen
            docker push guillemetal/app-hola-mundo:latest
          '''
        }
      }
    }

    stage('Desplegar en Kubernetes') {
      container('kubectl') {
        sh '''
          kubectl set image deployment/hola-mundo nginx=guillemetal/app-hola-mundo:latest -n default
        '''
      }
    }
  }
}