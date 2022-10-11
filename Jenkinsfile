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
            sh """
            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                              --context `pwd` \
                              --skip-tls-verify \
                              --destination=my-local.registry/nginx-test:${env.GIT_COMMIT.take(7)}
            """
          }
        }
      }
    }

/*
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
*/

    stage('Deploy to env') {
      steps {
        container('k8s') {
          script {
            dir ("${params.GIT_REPO}") {
              sh """
              kubectl config set-cluster k8s-transru --insecure-skip-tls-verify=true --server=`https://192.168.1.108:6443`
              kubectl config set-credentials git-ci --token=`eyJhbGciOiJSUzI1NiIsImtpZCI6IkdJVDNoZEJEZjgzSmp3NkZ4VmRDSk9udWprQWZDUXcxcDVKOExjTWZqTkkifQ`
              kubectl config set-context git-ci --cluster=k8s-transru --user=`jenkins-transru`
              kubectl config use-context git-ci
              kubectl get ns
              helm upgrade --install nginx-test .helm --namespace six --set registry=my-local.registry
              """
            }
          }
        }
      }
    }
  }
}


//    alpine/k8s:1.19.8
/*
kubectl config set-cluster k8s-transru --insecure-skip-tls-verify=true --server=${SERVER_URL}
kubectl config set-credentials git-ci --token=${SERVICE_ACCOUNT}
kubectl config set-context git-ci --cluster=k8s-transru --user=jenkins-transru
kubectl config use-context git-ci
*/
