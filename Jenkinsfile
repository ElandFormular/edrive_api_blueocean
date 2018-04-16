pipeline {
  agent any
  stages {
    stage('pull source and build') {
      steps {
        ansiColor(colorMapName: 'xterm') {
          timestamps() {
            node(label: 'edrive_api_node') {
              ws(dir: '/home/ec2-user/jenkins/workspace/edrive-api') {
                build '/edrive-api/01. edrive_api_build_source'
              }

            }

          }

        }

      }
    }
  }
}