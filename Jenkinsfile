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
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'master', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
      }
    }
    stage('junit test') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=prod test'''
      }
    }
    stage('build source') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=prod -Dmaven.test.skip=true package'''
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
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "/home/ec2-user/docker/source/production/"'
          }
        }
        stage('tag old image') {
          steps {
            sh '''docker rmi $ECR_REGISTRY:prd-old || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker tag $ECR_REGISTRY:prd $ECR_REGISTRY:prd-old || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker rmi $ECR_REGISTRY:prd || EXIT_CODE=$? && true ;
echo $EXIT_CODE
'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''cd $DOCKER_FILE
docker build -t $ECR_REGISTRY:prd --force-rm=false --pull=true -f ./edrive/Dockerfile ./
docker push $ECR_REGISTRY:prd'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
    DOCKER_FILE = '/home/ec2-user/docker'
    ECR_REGISTRY = '595483153913.dkr.ecr.ap-northeast-2.amazonaws.com/eland-dev-edrive-api/repo'
  }
}