pipeline {

    agent any

    environment {
        AWS_DEFAULT_REGION = "ap-south-1"
        TF_IN_AUTOMATION = "true"
    }
    
    parameters {
        choice(
             name: 'ACTION',
             choices: ['PLAN', 'APPLY', 'DESTROY'],
             description: 'Select Terraform action'
        )
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()

                git branch: 'main',
                    url: 'https://github.com/ManojBK18/Observability-end-to-end-Project.git'
            }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws_cred']
                ]) {
                    dir('terraform') {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Format Check') {
            steps {
                dir('terraform') {
                    sh 'terraform fmt -check'
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
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws_cred']
                ]) {

                    dir('terraform') {
                        sh '''
                            terraform plan \
                            -out=tfplan
                        '''
                    }

                }
            }
        }

        stage('Archive Plan') {
            steps {
                archiveArtifacts artifacts: 'terraform/tfplan',
                                 fingerprint: true
            }
        }

        stage('Terraform Apply') {
            when {
                expression { params.ACTION == 'APPLY' }
            }

            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws_cred']
                ]) {

                    input(
                        message: 'Apply Terraform Infrastructure?',
                        ok: 'Deploy'
                    )

                    dir('terraform') {
                        sh 'terraform apply -auto-approve tfplan'
                    }
                }
            }
        }
        stage('Terraform Destroy') {
            when {
                expression { params.ACTION == 'DESTROY' }
            }
        
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws_cred']
                ]) {

                    input(
                        message: 'Are you sure you want to destroy the infrastructure?',
                        ok: 'Destroy'
                    )

                    dir('terraform') {
                        sh 'terraform destroy -auto-approve'
                    }
                }
            }
        }

    }

    post {

        success {
            echo "Terraform deployment completed successfully."
        }

        failure {
            echo "Terraform deployment failed."
        }

        always {
            cleanWs()
        }

    }

    } 
