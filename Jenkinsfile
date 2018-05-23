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
            sh '''nextday="$(date --date="1 day" "+%FT%T.%N" | sed -r \'s/[[:digit:]]{7}$/Z/\')"

sudo chown -R ec2-user:ec2-user $WORKSPACE/
sudo chmod -R 755 $WORKSPACE/
rsync -avzh "${WORKSPACE}/deploy_scripts/" "${DEPLOY_SCRIPTS}/"
cp -rf "${DEPLOY_SCRIPTS}/was/setenv_prd.sh" "${SOURCE_DIR}/${BUILD_TYPE}/setenv.sh"
sed -i s/NEXTDATETIME/${nextday}/g "${DEPLOY_SCRIPTS}/aws/stage_instance_spec.json"

cat "${DEPLOY_SCRIPTS}/aws/stage_instance_spec.json"'''
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
      parallel {
        stage('delete untagged') {
          steps {
            sh '''IMAGES_TO_DELETE=$(aws ecr --profile $ECR_PROFILE_NAME list-images --repository-name $ECR_REPO --filter "tagStatus=UNTAGGED" --query \'imageIds[*]\' --output json )
aws ecr --profile $ECR_PROFILE_NAME batch-delete-image --repository-name $ECR_REPO --image-ids "$IMAGES_TO_DELETE" || true'''
          }
        }
        stage('create spot fleet') {
          steps {
            sh '''cd "${DEPLOY_SCRIPTS}/aws/"
aws ec2 --profile $PROFILE_NAME request-spot-fleet --spot-fleet-request-config file://stage_instance_spec.json'''
          }
        }
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.171-7.b10.37.amzn1.x86_64'
    M2_HOME = '/usr/share/apache-maven'
    ECR_REGISTRY = '595483153913.dkr.ecr.ap-northeast-2.amazonaws.com'
    BUILD_TYPE = 'prd'
    ECR_REPO = 'edrive-api-prd/repo'
    TAG = 'staging'
    DEPLOY_SCRIPTS = '/home/ec2-user/deploy_scripts'
    SOURCE_DIR = '/home/ec2-user/source'
    CODEDEPLOY_PATH = '/home/ec2-user/codedeploy'
    PROFILE_NAME = 'edrive-api-prd'
    ECR_PROFILE_NAME = 'edrive-api-dev'
  }
}