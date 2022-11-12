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
  environment {
    SERVER_URL = credentials('SERVER_URL')
    SERVICE_ACCOUNT = credentials('SERVICE_ACCOUNT')
  }
  stages {
    stage ( 'Kaniko build'){
      steps {
        container('kaniko'){
          script {
            sh "printenv"
            sh "echo ${env.GIT_COMMIT.take(7)}"
            sh "cat /etc/hosts"
            sh """
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                              --context `pwd` \
                              --skip-tls-verify \
                              --destination=docker-registry-service:5000/nginx-test:${env.GIT_COMMIT.take(7)}
            """
          }
        }
      }
    }


    stage('Deploy to env') {
      steps {
        container('helm-cli') {
          script {
            dir ("${params.GIT_REPO}") {
              sh """
              helm upgrade --install nginx-test .helm \
              --namespace jenkins-transru \
              --set registry=docker-registry-service:5000 \
              --set image.tag=${env.GIT_COMMIT.take(7)}
              """
            }
          }
        }
      }
    }
  }
}
