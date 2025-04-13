pipeline {
    agent {
    docker {
        image 'alpine/terraform:latest'
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
                    docker.image('hashicorp/terraform:1.5.7').inside {
                        dir('terraform') {
                            sh '''
                                terraform init
                                terraform validate
                                terraform plan -out=tfplan
                            '''
                        }

                        def applyApproval = input(
                            id: 'ApplyApproval',
                            message: 'Apply Terraform changes?',
                            parameters: [booleanParam(name: 'approve', defaultValue: false)]
                        )
                        if (!applyApproval) {
                            error("Terraform apply aborted by user")
                        }

                        dir('terraform') {
                            sh 'terraform apply -auto-approve tfplan'
                        }

                        def destroyApproval = input(
                            id: 'DestroyApproval',
                            message: 'Destroy Terraform infrastructure?',
                            parameters: [booleanParam(name: 'approve_destroy', defaultValue: false)]
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
    }

    post {
        always {
            cleanWs()
        }
    }
}
