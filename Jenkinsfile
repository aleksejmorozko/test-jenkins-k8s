properties([
  parameters(
    [
      stringParam(
        name: 'GIT_REPO',
        defaultValue: ''
      ),
      stringParam(
        name: 'VERSION',
        defaultValue: ''
      ),
      choiceParam(
        name: 'ENV',
        choices: ['test', 'staging', 'production']
      )
    ]
  )
])

pipeline {

  agent {
    kubernetes {
      yamlFile 'builder.yaml'
    }
  }

  stages {
/*    stage('Find short commit') {
      steps {
        container('git') {
          script {
            sh_commit = 'git rev-parse --short=8 HEAD'
            sh "echo $sh_commit"
          }
        }
      }
    }
*/
    stage ( 'Kaniko build'){
      steps {
        container('kaniko'){
          script {
            sh "printenv"
            sh "echo ${env.GIT_COMMIT.take(7)}"
            sh "echo '192.168.1.108 my-local.registry' >> /etc/hosts"
            sh "cat /etc/hosts"
            sh '''
            /kaniko/executor  --dockerfile `pwd`/Dockerfile \
                              --skip-tls-verify \
                              --destination=my-local.registry/nginx-test:${env.GIT_COMMIT.take(7)}
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
