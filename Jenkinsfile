podTemplate(
  label: 'jenkins-docker-agent',
  containers: [
    containerTemplate(
      name: 'docker',
      image: 'docker:24.0.7-dind',
      privileged: true,
      args: '--host tcp://0.0.0.0:2375',
      ttyEnabled: true,
      resourceRequestCpu: '100m',      // Solicita 0.1 CPU
      resourceLimitCpu: '200m',        // Límite máximo 0.2 CPU
      resourceRequestMemory: '256Mi',  // Solicita 256 MB RAM
      resourceLimitMemory: '512Mi'     // Límite máximo 512 MB RAM
    ),
    containerTemplate(
      name: 'docker-cli',
      image: 'docker:24.0.7-cli',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      resourceRequestCpu: '50m',       // Solicita 0.05 CPU
      resourceLimitCpu: '100m',        // Límite máximo 0.1 CPU
      resourceRequestMemory: '128Mi',  // Solicita 128 MB RAM
      resourceLimitMemory: '256Mi',    // Límite máximo 256 MB RAM
      envVars: [
        envVar(key: 'DOCKER_HOST', value: 'tcp://docker:2375')
      ],
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.docker', name: 'docker-config')
      ]
    ),
    containerTemplate(
      name: 'git',
      image: 'alpine/git',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      resourceRequestCpu: '50m',       // Solicita 0.05 CPU
      resourceLimitCpu: '100m',        // Límite máximo 0.1 CPU
      resourceRequestMemory: '128Mi',  // Solicita 128 MB RAM
      resourceLimitMemory: '256Mi',    // Límite máximo 256 MB RAM
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.ssh', name: 'git-ssh-key'),
        containerVolumeMount(mountPath: '/root/.ssh/known_hosts', name: 'ssh-known-hosts', subPath: 'known_hosts')
      ]
    ),
    containerTemplate(
      name: 'kubectl',
      image: 'bitnami/kubectl:latest',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      resourceRequestCpu: '50m',       // Solicita 0.05 CPU
      resourceLimitCpu: '100m',        // Límite máximo 0.1 CPU
      resourceRequestMemory: '128Mi',  // Solicita 128 MB RAM
      resourceLimitMemory: '256Mi',    // Límite máximo 256 MB RAM
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.kube', name: 'kube-config')
      ]
    )
  ],
  volumes: [
    secretVolume(secretName: 'docker-config', mountPath: '/root/.docker'),
    secretVolume(secretName: 'git-ssh-key', mountPath: '/root/.ssh'),
    secretVolume(secretName: 'ssh-known-hosts', mountPath: '/root/.ssh/known_hosts'),
    secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')
  ]
) {
  node('jenkins-docker-agent') {
    stage('Clonar Repo') {
      container('git') {
        sh '''
          mkdir -p /root/.ssh
          chmod 700 /root/.ssh
          cp /root/.ssh/known_hosts /root/.ssh/known_hosts  # Montado desde el Secret
          chmod 600 /root/.ssh/id_rsa
        '''
        git url: 'git@github.com:bonanza1958/app-hola-mundo.git'
      }
    }

    stage('Build & Push') {
      container('docker-cli') {
        sh '''
          docker login -u guillemetal --password-stdin <<< "dckr_pat_57TFqoPI5SiWR2Lxuv9vXphInPU"
          docker build -t guillemetal/app-hola-mundo:latest .
          docker push guillemetal/app-hola-mundo:latest
        '''
      }
    }

    stage('Desplegar') {
      container('kubectl') {
        sh '''
          kubectl set image deployment/hola-mundo nginx=guillemetal/app-hola-mundo:latest -n default
        '''
      }
    }
  }
}