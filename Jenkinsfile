pipeline {

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
            sh 'git rev-parse --short=8 HEAD'
            sh '''
            /kaniko/executor  --dockerfile `pwd`/Dockerfile \
                              --context `pwd` \
                              --destination=my-local.registry/nginx-test:${BUILD_NUMBER}
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
