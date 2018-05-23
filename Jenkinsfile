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
    stage('build source') {
      parallel {
        stage('build source') {
          steps {
            sh '''cd "$WORKSPACE/trunk/edrive-api/"
$M2_HOME/bin/mvn clean -Dspring.profiles.active=$BUILD_TYPE -Dmaven.test.skip=true package'''
          }
        }
        stage('prepare to scripts') {
          steps {
            sh '''rsync -avzh "${WORKSPACE}/deploy_scripts/" "${DEPLOY_SCRIPTS}/"
rsync -avzh "${DEPLOY_SCRIPTS}/was/setenv_dev.sh" "${SOURCE_DIR}/${BUILD_TYPE}/setenv.sh"'''
          }
        }
      }
    }
    stage('prepare to upload') {
      parallel {
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "${SOURCE_DIR}/${BUILD_TYPE}/"'
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
            sh '''getToken=$(aws --profile $PROFILE_NAME ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''cd "${SOURCE_DIR}/docker"
docker build -t $ECR_REGISTRY/$ECR_REPO:latest --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./edrive/Dockerfile ./'''
      }
    }
    stage('aws codedeploy') {
      steps {
        sh '''#!/bin/bash

APPLICATION_NAME=edrive-qd-fileservice
S3BUCKET=edrive-dev-codedeploy
S3FOLDER=fileservice-api-dev
S3EXTENSION=zip
S3FILE=${BUILD_NUMBER}-${BUILD_ID}.${S3EXTENSION}
DEPLOY_GROUP=$BUILD_TYPE
DEPLOY_CONFIG=CodeDeployDefault.AllAtOnce
SHUTDOWN_WAIT=600
DEPLOYMENT_ID=""
DEPLOY_STATUS=""

aws deploy --profile $PROFILE_NAME push --application-name $APPLICATION_NAME --s3-location "s3://${S3BUCKET}/${S3FOLDER}/${S3FILE}" --source "${CODEDEPLOY_PATH}"
DEPLOYMENT_ID=$(aws deploy --profile $PROFILE_NAME create-deployment --application-name $APPLICATION_NAME --deployment-group-name $DEPLOY_GROUP --deployment-config-name $DEPLOY_CONFIG --s3-location bundleType="${S3EXTENSION}",bucket="${S3BUCKET}",key="${S3FOLDER}/${S3FILE}" --query "deploymentId" --output text)

status(){
  if [ -n "$DEPLOYMENT_ID" ]
  then 
    DEPLOY_STATUS=$(aws deploy --profile $PROFILE_NAME get-deployment --deployment-id $DEPLOYMENT_ID --query "deploymentInfo.[status]" --output text)
  else
    echo -n -e "\\e[00;31mCodeDeploy is not running (Or Invalid deployment id)\\e[00m"
  fi
}

if [ -n "$DEPLOYMENT_ID" ]
then  
  echo -e "\\e[00;31mStart CodeDeploy : $DEPLOYMENT_ID\\e[00m"

  let wait=$SHUTDOWN_WAIT
  count=0;
  status
  until [ "$DEPLOY_STATUS" = "Succeeded" ] || [ "$DEPLOY_STATUS" = "Failed" ] ||[ $count -gt $wait ]
  do    
    sleep 1
    let count=$count+1;
    status
    echo -n -e "\\n\\e[00;31mwaiting for deploy processes : $DEPLOY_STATUS\\e[00m"
  done

  echo -n -e "\\n\\e[00;31mEnd CodeDeploy : $DEPLOY_STATUS\\e[00m"

  if [ "$DEPLOY_STATUS" != "Succeeded" ]
  then
    echo -n -e "ERROR CODE >>> "
    echo -n -e $(aws deploy --profile $PROFILE_NAME get-deployment --deployment-id $DEPLOYMENT_ID --query "deploymentInfo.errorInformation.code" --output text)
  fi
else
  echo -e "\\e[00;31mCodeDeploy is not running\\e[00m"
fi'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.37.amzn1.x86_64'
    M2_HOME = '/usr/share/apache-maven'
    DOCKER_FILE = '/home/ec2-user/docker'
    ECR_REGISTRY = 'noregistry'
    BUILD_TYPE = 'dev'
    ECR_REPO = 'eland-dev-edrive-api/repo'
    CONTAINER_NAME = 'edrive-api-dev'
    CODEDEPLOY_PATH = '/home/ec2-user/codedeploy'
    SOURCE_DIR = '/home/ec2-user/source'
    PROFILE_NAME = 'edrive-api-dev'
  }
  triggers {
    pollSCM('H/5 * * * *')
  }
}