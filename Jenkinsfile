pipeline {
  agent any
  stages {
    stage('pull source') {
      steps {
        echo 'start "pull source"'
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'develop', credentialsId: 'formularadmin')
        echo 'end "pull source"'
      }
    }
  }
}