pipeline {
  agent {
    node {
      label 'edrive_api_node'
      customWorkspace '/home/ec2-user/jenkins/workspace/edrive-api'
    }

  }
  stages {
    stage('pull source') {
      steps {
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'develop', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
      }
    }
    stage('junit test') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/bin/mvn clean -Dspring.profiles.active=$BUILD_TYPE test'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.37.amzn1.x86_64'
    M2_HOME = '/usr/share/apache-maven'
    BUILD_TYPE = 'dev'
  }
  triggers {
    pollSCM('H/5 * * * *')
  }
}