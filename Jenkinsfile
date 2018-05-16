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
            sh '''getToken=$(aws --profile edrive-api-dev ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''cd "$DOCKER_FILE"
docker build -t $ECR_REGISTRY/$ECR_REPO:latest --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./edrive/Dockerfile ./'''
      }
    }
    stage('aws codedeploy') {
      steps {
        sh '''PROFILE_NAME=edrive-api-dev
APPLICATION_NAME=edrive-qd-fileservice
S3BUCKET=edrive-dev-codedeploy
S3FOLDER=fileservice-api-dev
S3EXTENSION=zip
S3FILE=${BUILD_NUMBER}-${BUILD_ID}.${S3EXTENSION}
DEPLOY_GROUP=dev
DEPLOY_CONFIG=CodeDeployDefault.AllAtOnce


rsync -avz  "${WORKSPACE}/edrive-api/deploy_script/" "${CODEDEPLOY_PATH}/deploy_script/"
rsync -avz  "${WORKSPACE}/edrive-api/*.yml" "${CODEDEPLOY_PATH}/"
aws deploy --profile $PROFILE_NAME push --application-name $APPLICATION_NAME --s3-location "s3://${S3BUCKET}/fileservice-api-dev/#100-9020272551422205410.zip" --source "${CODEDEPLOY_PATH}"
aws deploy --profile $PROFILE_NAME create-deployment --application-name $APPLICATION_NAME --deployment-group-name $DEPLOY_GROUP --deployment-config-name $DEPLOY_CONFIG --s3-location bundleType="${S3EXTENSION}",bucket="${S3BUCKET}",key="${S3FOLDER}/${S3FILE}"'''
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
  }
  triggers {
    pollSCM('H/5 * * * *')
  }
}