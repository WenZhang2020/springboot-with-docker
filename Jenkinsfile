def label = "worker-${UUID.randomUUID().toString()}"

podTemplate(label: label, runAsUser: "0", runAsGroup: "0", containers: [
  containerTemplate(name: 'gradle', image: 'gradle:7.3.3-jdk11', command: 'cat', ttyEnabled: true),
  containerTemplate(name: 'kubectl', image: 'bitnami/kubectl', command: 'cat', ttyEnabled: true),
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
            withCredentials([string(credentialsId: 'DOCKER_HUB_PASSWORD', variable: 'PASSWORD')]) {
              sh """
                docker login -u wzhang12 -p '$PASSWORD'
                docker build -t jhooq-docker-demo .
                docker tag jhooq-docker-demo wzhang12/jhooq-docker-demo:jhooq-docker-demo
                docker push  wzhang12/jhooq-docker-demo:jhooq-docker-demo
                """
            }
        }
      }
   
    stage('Run kubectl') {
      container('kubectl') {
        sh """
            kubectl get all -n jenkins
            kubectl apply -f k8s-spring-boot-deployment.yml -n jenkins
            """
      }
    }
     }
    }
