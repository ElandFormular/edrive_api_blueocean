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
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/bin/mvn clean -Dspring.profiles.active=${BUILD_TYPE} -Dmaven.test.skip=true package'''
      }
    }
    stage('prepare to upload') {
      parallel {
        stage('login for aws') {
          steps {
            sh '''getToken=$(aws ecr --profile $ECR_PROFILE_NAME get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "${SOURCE_DIR}/${BUILD_TYPE}/"'
          }
        }
        stage('remove old image') {
          steps {
            sh '''docker rmi $ECR_REGISTRY/$ECR_REPO:${TAG} || EXIT_CODE=$? && true ;
echo $EXIT_CODE'''
          }
        }
        stage('prepare to scripts') {
          steps {
            sh '''sudo chown -R ec2-user:ec2-user $WORKSPACE/
sudo chmod -R 755 $WORKSPACE/
rsync -avzh "${WORKSPACE}/deploy_scripts/" "${DEPLOY_SCRIPTS}/"
cp -rf "${DEPLOY_SCRIPTS}/was/setenv_prd.sh" "${SOURCE_DIR}/${BUILD_TYPE}/setenv.sh"
cp -rf "${WORKSPACE}/deploy_scripts/codedeploy/start_container_prd.sh" "${WORKSPACE}/deploy_scripts/codedeploy/start_container.sh"
'''
          }
        }
      }
    }
    stage('create new image') {
      steps {
        sh '''cd "${DEPLOY_SCRIPTS}"
docker build -t $ECR_REGISTRY/$ECR_REPO:${TAG} --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./docker/edrive/Dockerfile ../
docker push $ECR_REGISTRY/$ECR_REPO:${TAG}'''
      }
    }
    stage('delete untagged') {
      steps {
        sh '''IMAGES_TO_DELETE=$(aws ecr --profile $ECR_PROFILE_NAME list-images --repository-name $ECR_REPO --filter "tagStatus=UNTAGGED" --query \'imageIds[*]\' --output json )
aws ecr --profile $ECR_PROFILE_NAME batch-delete-image --repository-name $ECR_REPO --image-ids "$IMAGES_TO_DELETE" || true'''
      }
    }
    stage('aws codedeploy') {
      steps {
        sh '''#!/bin/bash

APPLICATION_NAME=edrive-prd-fileservice
S3BUCKET=edrive-prd-codedeploy
S3FOLDER=fileservice-api-prd
S3EXTENSION=zip
S3FILE=${BUILD_NUMBER}-${BUILD_ID}.${S3EXTENSION}
DEPLOY_GROUP=$BUILD_TYPE
DEPLOY_CONFIG=CodeDeployDefault.OneAtATime
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
    ECR_REGISTRY = '595483153913.dkr.ecr.ap-northeast-2.amazonaws.com'
    ECR_REPO = 'edrive-api-prd/repo'
    TAG = '1.0'
    DEPLOY_SCRIPTS = '/home/ec2-user/deploy_scripts'
    SOURCE_DIR = '/home/ec2-user/source'
    CODEDEPLOY_PATH = '/home/ec2-user/codedeploy'
    PROFILE_NAME = 'edrive-api-prd'
    ECR_PROFILE_NAME = 'edrive-api-dev'
  }
}