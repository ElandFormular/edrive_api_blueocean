pipeline {
  agent any
  stages {
    stage('pull source') {
      steps {
        ansiColor(colorMapName: 'xterm') {
          timestamps() {
            node(label: 'edrive_api_node') {
              ws(dir: '/home/ec2-user/jenkins/workspace/edrive-api') {
                git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'develop', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
              }

            }

          }

        }

      }
    }
    stage('build') {
      steps {
        node(label: 'edrive_api_node') {
          ws(dir: '/home/ec2-user/jenkins/workspace/edrive-api') {
            sh '''echo $M2
cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Pdev -Dmaven.test.skip=true package'''
          }

        }

      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
  }
}