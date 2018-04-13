pipeline {
  agent any
  stages {
    stage('build source') {
      steps {
        build(job: '/edrive-api/01. edrive_api_build_source', propagate: true)
      }
    }
    stage('make base image') {
      steps {
        build(job: '/edrive-api/02. edrive_api_image_base', propagate: true)
      }
    }
    stage('make develop image') {
      steps {
        build(job: '/edrive-api/03. edrive_api_image_develop', propagate: true)
      }
    }
  }
}