podTemplate(
  label: 'jenkins-docker-agent',
  containers: [
    containerTemplate(
      name: 'docker',
      image: 'docker:24.0.7-dind',
      privileged: true,
      args: '--host tcp://0.0.0.0:2375',
      ttyEnabled: true,
      resourceRequestCpu: '500m',
      resourceLimitCpu: '1000m',
      resourceRequestMemory: '512Mi',
      resourceLimitMemory: '1Gi'
    ),
    containerTemplate(
      name: 'docker-cli',
      image: 'docker:24.0.7-cli',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      envVars: [
        envVar(key: 'DOCKER_HOST', value: 'tcp://docker:2375')
      ],
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.docker', name: 'docker-config')  // Sintaxis correcta
      ]
    ),
    containerTemplate(
      name: 'git',
      image: 'alpine/git',
      command: 'sleep',
      args: 'infinity',
      ttyEnabled: true,
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.ssh', name: 'ssh-keys')  // Sintaxis correcta
      ]
    ),
    containerTemplate(
      name: 'kubectl',
      image: 'bitnami/kubectl:latest',
      command: 'sleep',
      args: 'infinity',
      volumeMounts: [
        containerVolumeMount(mountPath: '/root/.kube', name: 'kube-config')  // Sintaxis correcta
      ]
    )
  ],
  volumes: [
    secretVolume(secretName: 'docker-config', mountPath: '/root/.docker'),
    secretVolume(secretName: 'ssh-keys', mountPath: '/root/.ssh'),
    secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')
  ]
) {
  node('jenkins-docker-agent') {
    // ... etapas del pipeline
  }
}