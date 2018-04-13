pipeline {
  agent any
  stages {
    stage('build source') {
      steps {
        build '/edrive-api/01. edrive_api_build_source'
      }
    }
    stage('make base image') {
      steps {
        build '/edrive-api/02. edrive_api_image_base'
      }
    }
    stage('make develop image') {
      steps {
        build '/edrive-api/03. edrive_api_image_develop'
      }
    }
  }
}