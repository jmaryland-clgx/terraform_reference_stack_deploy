pipeline {
    agent {
        dockerfile {
            filename 'Dockerfile'
            additionalBuildArgs "--build-arg terraform_version=0.11.14"
        }
    }
    stages {
        stage('Lint') {
            steps {
                sh 'terraform version'
                sh 'terraform fmt -check' // fail build on lint corrections
                sshagent(['testauth']) {
                    sh 'echo SSH_AUTH_SOCK=$SSH_AUTH_SOCK'
                    sh 'ls -al $SSH_AUTH_SOCK || true'
                    sh 'ssh -vvv -o StrictHostKeyChecking=no git.epam.com uname -a || true'
                    sh 'terraform init -input=false -backend=false' //dont try to connect to a backend yet
                }
                sh 'terraform validate'
            }
        }
        stage('Sandbox') { // Deploy to default backend/state using default workspace
            steps {
                //when {
                //    branch 'develop' // Optional for git flow
                //}
                sh 'echo "replace me with default credentials for GOOGLE_CREDENTIALS"'
                // or GOOGLE_CLOUD_KEYFILE_JSON
                // or  GCLOUD_KEYFILE_JSON
                // or GOOGLE_APPLICATION_CREDENTIALS'
                sshagent(['testauth']) {
                    sh 'echo SSH_AUTH_SOCK=$SSH_AUTH_SOCK'
                    sh 'ls -al $SSH_AUTH_SOCK || true'
                    sh 'ssh -vvv -o StrictHostKeyChecking=no git.epam.com uname -a || true'
                    sh 'terraform init -input=false'
                }
                sh 'terraform workspace select default' // or sandbox, dev, etc.
                sh 'terraform plan -input=false .terraform/plan'
                sh 'terraform apply -input=false .terraform/plan'
                sh 'echo "should run validator here. Terratest, inspec, terraform-compliance, etc"'
            }
        }
        stage('Prod') {  // Deploy to prod backend/state using prod workspace
            steps {
                when {
                    branch 'master'
                }
                sh 'echo "replace me with prod credentials for GOOGLE_CREDENTIALS"'
                sshagent(['testauth']) {
                    sh 'echo SSH_AUTH_SOCK=$SSH_AUTH_SOCK'
                    sh 'ls -al $SSH_AUTH_SOCK || true'
                    sh 'ssh -vvv -o StrictHostKeyChecking=no git.epam.com uname -a || true'
                    sh 'terraform init -input=false -backend-config=prod.tfbackend' // use a production statefile
                }
                sh 'terraform workspace select prod'
                sh 'terraform plan -input=false .terraform/plan' // optional: --var-file prod.tvars
                input "Deploy to prod?"
                sh 'echo "should run validator here. Terratest, inspec, terraform-compliance, etc"'
            }
        }
    }
}