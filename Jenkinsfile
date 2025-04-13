pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Saiketha/Terraform-Jenkins.git'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Validate') {
            steps {
                sh 'terraform validate'
            }
        }

        stage('Terraform Plan') {
            steps {
                sh 'terraform plan -out=tfplan'
            }
        }

        stage('Approval to Apply') {
            steps {
                script {
                    def applyApproval = input(
                        id: 'ApplyApproval', message: 'Apply Terraform changes?', parameters: [
                            [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Check to approve', name: 'approve']
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
                sh 'terraform apply -auto-approve tfplan'
            }
        }

        stage('Approval to Destroy') {
            steps {
                script {
                    def destroyApproval = input(
                        id: 'DestroyApproval', message: 'Destroy Terraform infrastructure?', parameters: [
                            [$class: 'BooleanParameterDefinition', defaultValue: false, description: 'Check to approve destroy', name: 'approve_destroy']
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
                sh 'terraform destroy -auto-approve'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
