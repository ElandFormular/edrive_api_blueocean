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
      steps {
        sh '''cd "$WORKSPACE/trunk/edrive-api/"
$M2_HOME/bin/mvn clean -Dspring.profiles.active=$BUILD_TYPE -Dmaven.test.skip=true package'''
      }
    }
    stage('prepare to upload') {
      parallel {
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "${SOURCE_DIR}/${BUILD_TYPE}/"'
          }
        }
        stage('docker login') {
          steps {
            sh '''getToken=$(aws ecr --profile $ECR_PROFILE_NAME get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
        stage('volume permission') {
          steps {
            sh 'sudo chmod -R 755 $VOLUME_DIR/'
          }
        }
        stage('prepare to scripts') {
          steps {
            sh '''sudo chown -R ec2-user:ec2-user $WORKSPACE/
sudo chmod -R 755 $WORKSPACE/
rsync -avzhr --delete "${WORKSPACE}/deploy_scripts/" "${DEPLOY_SCRIPTS}/"
cp -rf "${DEPLOY_SCRIPTS}/codedeploy/appspec.dev.yml" "${DEPLOY_SCRIPTS}/codedeploy/appspec.yml"
cp -rf "${DEPLOY_SCRIPTS}/was/setenv_dev.sh" "${SOURCE_DIR}/${BUILD_TYPE}/setenv.sh"'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''cd "${DEPLOY_SCRIPTS}"
docker build -t $ECR_REGISTRY/$ECR_REPO:latest --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./docker/edrive/Dockerfile ../'''
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

aws deploy --profile $PROFILE_NAME push --application-name $APPLICATION_NAME --s3-location "s3://${S3BUCKET}/${S3FOLDER}/${S3FILE}" --source "${DEPLOY_SCRIPTS}/codedeploy"
DEPLOYMENT_ID=$(aws deploy --profile $PROFILE_NAME create-deployment --application-name $APPLICATION_NAME --deployment-group-name $DEPLOY_GROUP --deployment-config-name $DEPLOY_CONFIG --s3-location bundleType="${S3EXTENSION}",bucket="${S3BUCKET}",key="${S3FOLDER}/${S3FILE}" --query "deploymentId" --output text)

status(){
  if [ -n "$DEPLOYMENT_ID" ]
  then 
    DEPLOY_STATUS=$(aws deploy --profile $PROFILE_NAME get-deployment --deployment-id $DEPLOYMENT_ID --query "deploymentInfo.[status]" --output text)
  else
    echo -n -e "\\e[00;31mCodeDeploy is not running (Or Invalid deployment id)\\e[00m"
    exit 1
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
    exit 1
  fi
else
  echo -e "\\e[00;31mCodeDeploy is not running\\e[00m"
  exit 1
fi'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.37.amzn1.x86_64'
    M2_HOME = '/usr/share/apache-maven'
    DEPLOY_SCRIPTS = '/home/ec2-user/deploy_scripts'
    ECR_REGISTRY = 'noregistry'
    BUILD_TYPE = 'dev'
    ECR_REPO = 'edrive-api-dev/repo'
    CODEDEPLOY_PATH = '/home/ec2-user/codedeploy'
    SOURCE_DIR = '/home/ec2-user/source'
    PROFILE_NAME = 'edrive-api-dev'
    VOLUME_DIR = '/home/ec2-user/volume'
    ECR_PROFILE_NAME = 'edrive-api-dev'
  }
  triggers {
    pollSCM('H/5 * * * *')
  }
}