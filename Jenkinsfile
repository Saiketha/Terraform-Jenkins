pipeline {
    agent {
        docker {
            image 'hashicorp/terraform:1.5.7'
            args '--entrypoint=""'
        }
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Run Terraform in Docker') {
            steps {
                script {
                    // Run Terraform commands inside the docker container
                    dir('terraform') {
                        sh '''
                            terraform init
                            terraform validate
                            terraform plan -out=tfplan
                        '''
                    }

                    // Approval to apply changes
                    def applyApproval = input(
                        id: 'ApplyApproval',
                        message: 'Apply Terraform changes?',
                        parameters: [
                            booleanParam(name: 'approve', defaultValue: false, description: 'Approve Apply')
                        ]
                    )
                    if (!applyApproval) {
                        error("Terraform apply aborted by user")
                    }

                    // Apply Terraform changes
                    dir('terraform') {
                        sh 'terraform apply -auto-approve tfplan'
                    }

                    // Approval to destroy infrastructure
                    def destroyApproval = input(
                        id: 'DestroyApproval',
                        message: 'Destroy Terraform infrastructure?',
                        parameters: [
                            booleanParam(name: 'approve_destroy', defaultValue: false, description: 'Approve Destroy')
                        ]
                    )
                    if (destroyApproval) {
                        dir('terraform') {
                            sh 'terraform destroy -auto-approve'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Cleanup workspace
        }
    }
}
