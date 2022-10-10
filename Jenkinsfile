pipeline {
  options {
    ansiColor('xterm')
  }

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {

    stage ( 'Kaniko build'){
      steps {
        container('kaniko'){
          script {
            sh '''
            /kaniko/executor  --dockerfile ./Dockerfile \
                              --context `pwd` \
                              --destination=my-local.registry/nginx-test:${git rev-parse --short=8 HEAD}
            '''
          }
        }
      }
    }

    stage('deploy'){
      steps {
        container('kubectl'){
          withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
            sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
            sh 'kubectl apply -f myweb.yaml'
          }
        }
      }
    }
  }
}
