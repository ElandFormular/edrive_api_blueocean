pipeline {
  agent {
    node {
      label 'edrive_api_node'
      customWorkspace '/home/ec2-user/jenkins/workspace/edrive-api'
    }

  }

  triggers {
    pollSCM('H/5 * * * *')
  }
  stages {
    stage('pull source') {
      steps {
        git(url: 'http://10.123.180.232:8090/scm/git/2017/edrive_Api_Project', branch: 'develop', credentialsId: '347d447d-ae19-4798-84c0-cfa598960058')
      }
    }
    stage('junit test') {
      steps {
        sh '''cd $WORKSPACE/trunk/edrive-api/
$M2_HOME/mvn clean -Dspring.profiles.active=$BUILD_TYPE test'''
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
        stage('login for aws') {
          steps {
            sh '''getToken=$(aws ecr get-login --no-include-email --region ap-northeast-2)

getLogin=$($getToken)

echo $get-login'''
          }
        }
        stage('move war file') {
          steps {
            sh 'cp -rf "${WORKSPACE}/trunk/edrive-api/target/ROOT.war" "/home/ec2-user/docker/source/${BUILD_TYPE}/"'
          }
        }
        stage('tag old image') {
          steps {
            sh '''docker rmi $ECR_REGISTRY:${BUILD_TYPE}-old || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker tag $ECR_REGISTRY:${BUILD_TYPE} $ECR_REGISTRY:${BUILD_TYPE}-old || EXIT_CODE=$? && true ;
echo $EXIT_CODE

docker rmi $ECR_REGISTRY:${BUILD_TYPE} || EXIT_CODE=$? && true ;
echo $EXIT_COD
'''
          }
        }
      }
    }
    stage('prepare to new') {
      parallel {
        stage('create image') {
          steps {
            sh '''cd $DOCKER_FILE
docker build -t $ECR_REGISTRY:${BUILD_TYPE} --force-rm=false --pull=true --build-arg BUILD_TYPE=$BUILD_TYPE -f ./edrive/Dockerfile ./
docker push $ECR_REGISTRY:${BUILD_TYPE}'''
          }
        }
        stage('stop container') {
          steps {
            sh '''docker stop edrive-api-dev
docker rm -f edrive-api-dev'''
          }
        }
      }
    }
    stage('start container') {
      steps {
        sh 'docker run -it -d -p 8080:8080 --name edrive-api-dev $ECR_REGISTRY:dev'
      }
    }
    stage('start tomcat') {
      steps {
        sh 'docker exec -i edrive-api-dev /usr/local/tomcat/bin/catalina.sh start'
      }
    }
  }
  environment {
    JAVA_HOME = '/usr/lib/jvm/java-1.8.0-openjdk.x86_64'
    M2_HOME = '/usr/local/maven/bin'
    DOCKER_FILE = '/home/ec2-user/docker'
    ECR_REGISTRY = '595483153913.dkr.ecr.ap-northeast-2.amazonaws.com/eland-dev-edrive-api/repo'
    BUILD_TYPE = 'dev'
  }

}