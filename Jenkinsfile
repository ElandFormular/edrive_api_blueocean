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
    stage('prepare to upload') {
      parallel {
        stage('prepare to upload') {
          steps {
            node(label: 'edrive_api_node') {
              ws(dir: '/home/ec2-user/jenkins/workspace/edrive-api') {
                sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "/home/ec2-user/source/develop/"'
              }

            }

          }
        }
        stage('login for ecr') {
          steps {
            node(label: 'edrive_api_node') {
              ws(dir: '/home/ec2-user/jenkins/workspace/edrive-api') {
                sh '''getToken=$(aws ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
              }

            }

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