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
    stage('test source') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=dev test'''
      }
    }
    stage('build source') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=dev -Dmaven.test.skip=true package'''
      }
    }
    stage('prepare to upload') {
      parallel {
        stage('login for aws') {
          steps {
            sh '''getToken=$(aws ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "/home/ec2-user/docker/source/develop/"'
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''ECR_REGISTRY=95483153913.dkr.ecr.ap-northeast-2.amazonaws.com/eland-dev-edrive-api/repo
cd $DOCKER_FILE
docker build -t $ECR_REGISTRY:dev --pull=true -f ./edrive/Dockerfile ./
docker push $ECR_REGISTRY:dev'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
    DOCKER_FILE = '/home/ec2-user/docker'
  }
}