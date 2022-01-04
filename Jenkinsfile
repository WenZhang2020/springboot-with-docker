def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, runAsUser: "0", runAsGroup: "0", containers: [
  containerTemplate(name: 'gradle', image: 'gradle:7.3.3-jdk11', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'dockerindocker', image: 'aimvector/jenkins-slave', command: 'sleep', args: '99d')
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
        sh "gradle build"
      }
    }
    stage('Create images') {
      container('dockerindocker') {
          sh """
            docker build -t jhooq-docker-demo .
            """
        }
      }
    }
    
    }
