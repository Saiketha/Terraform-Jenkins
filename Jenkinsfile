pipeline {
    agent {
        docker {
            image 'hashicorp/terraform:1.5.7' // Use appropriate Terraform version
            args '-u root' // Optional: ensures permissions if workspace needs write
        }
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')        // Jenkins credentials ID
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')    // Jenkins credentials ID
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Saiketha/Terraform-Jenkins.git'
            }
        }

        stage('Terraform Init') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                dir('terraform') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Terraform Plan') {
            steps {
                dir('terraform') {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }

        stage('Approval to Apply') {
            steps {
                script {
                    def applyApproval = input(
                        id: 'ApplyApproval',
                        message: 'Apply Terraform changes?',
                        parameters: [
                            booleanParam(name: 'approve', defaultValue: false, description: 'Check to approve')
                        ]
                    )

                    if (!applyApproval) {
                        error("Terraform apply aborted by user")
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('terraform') {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }

        stage('Approval to Destroy') {
            steps {
                script {
                    def destroyApproval = input(
                        id: 'DestroyApproval',
                        message: 'Destroy Terraform infrastructure?',
                        parameters: [
                            booleanParam(name: 'approve_destroy', defaultValue: false, description: 'Check to approve destroy')
                        ]
                    )

                    if (!destroyApproval) {
                        echo "Destroy skipped by user"
                        currentBuild.result = 'SUCCESS'
                        return
                    }
                }
            }
        }

        stage('Terraform Destroy') {
            steps {
                dir('terraform') {
                    sh 'terraform destroy -auto-approve'
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
