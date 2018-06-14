def AWS_CREDENTIALS_ID = 'b18defc6-8872-41ca-a283-afc64f68b857'

pipeline {
  agent {
    docker {
      image 'jenkins201/hashicorp-ci:latest'
    }
  }

  environment {
     AWS_DEFAULT_REGION = 'eu-west-1'
  }

  stages {
    stage('Terraform Plan') {
      steps {
        checkout scm
        echo "My branch is: ${env.BRANCH_NAME}"

        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "mkdir plan"
            sh "terraform init"
            sh "terraform plan -out=plan/plan.out"
          }
        }
      }
    }

    stage('Manual Approval') {
      // TODO: this should be outside the implicit node definition, but then we'd
      // have to work out how to manage the plan/plan.out being persisted between stages
      // (probably use stash & unstash?)
      input 'Do you approve the apply?'
    }

    stage('Terraform Apply') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'demo-aws-creds',
                          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY' ]]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "terraform init"
            sh "terraform apply plan/plan.out"
          }
        }
      }
    }
  }
}
