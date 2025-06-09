pipeline {
  agent any

  environment {
    JF_URL = credentials('jfrog-url')
    JF_USER = credentials('jfrog-user')
    JF_PASSWORD = credentials('jfrog-password')
  }

  stages {
    stage('Clone Repo') {
      steps {
        checkout scm
      }
    }

    stage('Run Frogbot') {
      steps {
        sh '''
          curl -fL https://releases.jfrog.io/artifactory/frogbot/v2/frogbot-linux-amd64 -o frogbot
          chmod +x frogbot
          ./frogbot scan-pull-request
        '''
      }
    }
  }
}
