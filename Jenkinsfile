pipeline {
  agent any
  stages {
    stage('setting') {
      steps {
        node(label: 'edrive_api_node')
        ws(dir: '/home/ec2-user/jenkins/workspace/edrive_api')
        ansiColor(colorMapName: 'xtem')
      }
    }
    stage('pull source') {
      steps {
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'develop', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
      }
    }
    stage('build') {
      steps {
        build 'edirve-api/edrive_api_build'
      }
    }
  }
}