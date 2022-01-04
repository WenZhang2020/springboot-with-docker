def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, runAsUser: "0", runAsGroup: "0", containers: [
  containerTemplate(name: 'gradle', image: 'gradle:4.5.1-jdk9', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'golang', image: 'golang:1.16.5', command: 'sleep', args: '99d')
],volumes: [
  hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
]) {
  node(label) {
    def myRepo = checkout scm
    def gitCommit = myRepo.GIT_COMMIT
    def gitBranch = myRepo.GIT_BRANCH
    def shortGitCommit = "${gitCommit[0..10]}"
    def previousGitCommit = sh(script: "git rev-parse ${gitCommit}~", returnStdout: true)
    
    stage('Test') {
      try {
        container('gradle') {
          sh """
            pwd
            whoami
            id -u
            echo "GIT_BRANCH=${gitBranch}" >> /etc/environment
            echo "GIT_COMMIT=${gitCommit}" >> /etc/environment
            gradle test
            """
        }
      }
      catch (exc) {
        println "Failed to test - ${currentBuild.fullDisplayName}"
        throw(exc)
      }
    }
    stage('Build') {
      container('gradle') {
        sh """
          git clone https://github.com/WenZhang2020/springboot-with-docker.git
          cd springboot-with-docker
          ./gradlew build
        """
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
