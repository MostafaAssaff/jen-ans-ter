pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Check Terraform files') {
            steps {
                dir('Terraform') {
                    sh 'ls -al'
                }
            }
        }

        stage('Terraform Init') {
            steps {
                dir('Terraform') {
                    withCredentials([usernamePassword(credentialsId: 'aws-cred',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                            sh '''
                                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                                terraform init
                            '''
                    }
                }
            }
        }

        stage('Terraform Apply') {
            steps {
                dir('Terraform') {
                    withCredentials([usernamePassword(credentialsId: 'aws-cred',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                            sh '''
                                terraform apply -auto-approve \
                                  -var "aws_access_key=$AWS_ACCESS_KEY_ID" \
                                  -var "aws_secret_key=$AWS_SECRET_ACCESS_KEY"
                            '''
                    }
                }
            }
        }

        stage('Run Ansible') {
            steps {
                dir('Ansible') {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER')]) {
                        sh '''
                            EC2_IP=$(terraform -chdir=../Terraform output -raw public_ip)
                            echo "$EC2_IP ansible_user=$SSH_USER ansible_ssh_private_key_file=$SSH_KEY" > inventory
                            ansible-playbook -i inventory playbook.yml
                        '''
                    }
                }
            }
        }
    }
}
