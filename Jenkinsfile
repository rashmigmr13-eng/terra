pipeline {
    agent any

    parameters {
        choice(
            name: 'terraformAction',
            choices: ['apply', 'destroy'],
            description: 'Choose your Terraform action (apply or destroy)'
        )
    }

    environment {
        AWS_ACCESS_KEY_ID      = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {

        stage('Git Checkout') {
            steps {
                dir('terraform') {
                    git branch: 'main', url: 'https://github.com/rashmigmr13-eng/terra.git'
                }
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                dir('terraform/project-1') {
                    sh '''
                        echo "Initializing Terraform..."
                        terraform init

                        echo "Running Terraform Plan..."
                        terraform plan -out=tfplan

                        echo "Saving plan output..."
                        terraform show -no-color tfplan > tfplan.txt
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                script {
                    def planOutput = readFile('terraform/project-1/tfplan.txt')

                    input(
                        message: "Do you approve this Terraform run?",
                        parameters: [
                            text(
                                name: 'Terraform Plan Output',
                                defaultValue: planOutput,
                                description: 'Review this plan before approving'
                            )
                        ]
                    )
                }
            }
        }

        stage('Terraform Apply / Destroy') {
            when {
                expression {
                    params.terraformAction == 'apply' || params.terraformAction == 'destroy'
                }
            }

            steps {
                dir('terraform/project-1') {
                    script {

                        if (params.terraformAction == 'apply') {
                            echo "Applying Terraform changes..."
                            sh 'terraform apply -input=false tfplan'
                        } else {
                            echo "Destroying Terraform infrastructure..."
                            sh 'terraform destroy -auto-approve'
                        }

                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Terraform pipeline completed successfully!'
        }
        failure {
            echo '❌ Terraform pipeline failed — check logs!'
        }
        always {
            echo '📦 Pipeline execution finished.'
        }
    }
}
