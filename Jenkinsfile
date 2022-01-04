def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, containers: [
  containerTemplate(name: 'gradle', image: 'gradle:4.5.1-jdk9', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'golang', image: 'golang:1.16.5', command: 'sleep', args: '99d')
],volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    stage('Build') {
      container('gradle') {
        sh "./gradlew build"
      }
    }
    stage('Create images') {
      container('golang') {
          sh """
            docker build -t jhooq-docker-demo .
            """
        }
      }
    }
    
    }
