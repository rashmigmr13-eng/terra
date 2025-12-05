pipeline {
    agent any

    parameters {
https://github.com/rashmigmr13-eng/terra.git        choice(name: 'terraformAction', choices: ['apply', 'destroy'], description: 'Choose your Terraform action to perform')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

    stages {
        stage('Git Checkout') {
            steps {
                dir('terraform') {
                    git branch: 'terraform', url: 'https://github.com/rashmigmr13-eng/terra.git'
                }
            }
        }

        stage('Terraform Init & Plan') {
            steps {
                dir('terraform/project-1') {
                    sh '''
                        echo "Initializing Terraform..."
                        terraform init

                        echo "Running Terraform plan..."
                        terraform plan -out=tfplan

                        echo "Generating human-readable plan output..."
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
                        message: "Do you want to proceed with the Terraform action?",
                        parameters: [
                            text(name: 'Terraform Plan Output', defaultValue: planOutput, description: 'Review the plan before continuing.')
                        ]
                    )
                }
            }
        }

        stage('Terraform Apply or Destroy') {
            when {
                expression {
                    return params.terraformAction == 'apply' || params.terraformAction == 'destroy'
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
            echo '✅ Terraform action completed successfully!'
        }
        failure {
            echo '❌ Terraform action failed. Please check the logs.'
        }
        always {
            echo '📦 Pipeline execution completed.'
        }
    }
}
