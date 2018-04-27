pipeline {
  agent {
    node {
      label 'edrive_api_node'
      customWorkspace '/home/ec2-user/jenkins/workspace/edrive-api'
    }

  }
  stages {
    stage('prepare to upload') {
      parallel {
        stage('login for aws') {
          steps {
            sh '''getToken=$(aws ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
        stage('tag old image') {
          steps {
            sh '''docker rmi $ECR_REGISTRY/$ECR_REPO:latest || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker tag $ECR_REGISTRY/$ECR_REPO:staging $ECR_REGISTRY/$ECR_REPO:latest || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker tag $ECR_REGISTRY/$ECR_REPO:latest $ECR_REGISTRY/$ECR_REPO:$TAG || EXIT_CODE=$? && true ;
echo $EXIT_CODE'''
          }
        }
      }
    }
    stage('create image') {
      steps {
        sh '''docker push $ECR_REGISTRY/$ECR_REPO:latest
docker push $ECR_REGISTRY/$ECR_REPO:$TAG'''
      }
    }
    stage('delete untagged') {
      steps {
        sh '''IMAGES_TO_DELETE=$( aws ecr list-images --repository-name $ECR_REPO --filter "tagStatus=UNTAGGED" --query \'imageIds[*]\' --output json )
aws ecr batch-delete-image --repository-name $ECR_REPO --image-ids "$IMAGES_TO_DELETE" || true'''
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
    DOCKER_FILE = '/home/ec2-user/docker'
    ECR_REGISTRY = '024204912394.dkr.ecr.ap-northeast-2.amazonaws.com'
    BUILD_TYPE = 'prod'
    ECR_REPO = 'edrive-api-prd/repo'
    TAG = '1.0'
  }
}