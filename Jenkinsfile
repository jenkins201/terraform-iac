
// cdba6976-168f-4d52-bbbe-e65bec08ab89 = AWS DEV
// b18defc6-8872-41ca-a283-afc64f68b857 = AWS INT
def AWS_CREDENTIALS_ID = 'b18defc6-8872-41ca-a283-afc64f68b857'

pipeline {
  // currently only our master has python-virtualenv :(
  agent { node { label 'master' } }

  environment {
     AWS_DEFAULT_REGION = 'eu-west-1'
  }

  // due to the use of /tmp/ansible_tmp_deployment_vars.yml, limit to a single pipeline at a time
  options {
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }

  stages {
    // clone the primary repo & run the self contained setup.sh
    stage('Workspace setup') {
      steps {
        // wipe the workspace before we start
        deleteDir()

        echo "My branch is: ${env.BRANCH_NAME}"

        // notify stash/bitbucket that a build is in progress
        step([$class: 'StashNotifier'])

        dir('cloud_infra_int') {
          checkout scm
        }
        dir('configurationmanagement') {
          git url: 'http://home.taskize.co.uk/stash/scm/ops/configurationmanagement.git',
              credentialsId: 'd1640332-2c44-47e3-942f-e28bf480708e',
              branch: 'INFRA-591-Add-check-mode-to-provision.sh'
        }
        wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
          sh "cd configurationmanagement ; ./setup.sh --skip-repo-clone"
        }
      }
    }

    stage ('Terraform Branch Sanity Check') {
      when {
        expression { env.BRANCH_NAME != 'master' }
      }
      steps {
        echo "Hello, non-master branch!"
        // this is where we should sanity check the change - how?
        // the best we can do is run `provision.sh --check` but ignore the
        // failure and use the plan as a review point

        // Expose the AWS Int IAM credentials
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: AWS_CREDENTIALS_ID,
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID']]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "cd cloud_infra_int ; . ../configurationmanagement/.venv/bin/activate ; terraform init"
            // POC - should we give jenkins a gpg key to git-crypt unlock?
            sh "cd cloud_infra_int ; rm certificates.tf"
            sh "cd cloud_infra_int ; ../configurationmanagement/tools/igniter/provision.sh --check || true"
          }
        }
      }
    }

    stage('Terraform Drift Check - POC') {
      when {
        // Only say hello if a "greeting" is requested
        expression { env.BRANCH_NAME == 'master' }
      }
      steps {
        // TODO: check for drift - email ops on job failure
        // TODO: decide approach on secrets management
        //
        // Expose the AWS Int IAM credentials
        withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: AWS_CREDENTIALS_ID,
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID']]) {
          wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {
            sh "cd cloud_infra_int ; . ../configurationmanagement/.venv/bin/activate ; terraform init"
            // POC - should we give jenkins a gpg key to git-crypt unlock?
            sh "cd cloud_infra_int ; rm certificates.tf"
            sh "cd cloud_infra_int ; ../configurationmanagement/tools/igniter/provision.sh --check"
          }
        }
      }
    }

  }

  post {
    success {
      // set the build status & notify stash/bitbucket
      script { currentBuild.result = 'SUCCESS' }
      step([$class: 'StashNotifier'])
    }
    failure {
      // set the build status & notify stash/bitbucket
      script { currentBuild.result = 'FAILURE' }
      step([$class: 'StashNotifier'])
    }
  }
}
