pipeline {
  agent {
    node {
      label 'edrive_api_node'
      customWorkspace '/home/ec2-user/jenkins/workspace/edrive-api/02. edrive_api_deploy_dev'
    }
  }
  stages {
    stage('pull source') {
      steps {
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'master', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
      }
    }
    stage('build source') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=$BUILD_TYPE -Dmaven.test.skip=true package'''
      }
    }
    stage('prepare to upload') {
      parallel {
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "/home/ec2-user/docker/source/${BUILD_TYPE}/"'
          }
        }
        stage('tag old image') {
          steps {
            sh '''docker rmi $ECR_REGISTRY/$ECR_REPO:latest || EXIT_CODE=$? && true ;
echo $EXIT_COD
'''
          }
        }
        stage('docker login') {
          steps {
            sh '''getToken=$(aws --profile edrive-dev ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''cd $DOCKER_FILE
docker build -t $ECR_REGISTRY/$ECR_REPO:latest --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./edrive/Dockerfile ./'''
      }
    }
    stage('aws codedeploy') {
      steps {
        build '/edrive-api/02. edrive_api_deploy_dev'
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
    DOCKER_FILE = '/home/ec2-user/docker'
    ECR_REGISTRY = 'noregistry'
    BUILD_TYPE = 'dev'
    ECR_REPO = 'eland-dev-edrive-api/repo'
    CONTAINER_NAME = 'edrive-api-dev'
  }
  triggers {
    pollSCM('H/5 * * * *')
  }
}