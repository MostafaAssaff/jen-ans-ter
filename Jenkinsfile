pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
    }

    stages {
        stage('Terraform Init') {
            steps {
                dir('terraform') {
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
                dir('terraform') {
                    withCredentials([usernamePassword(credentialsId: 'aws-cred',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                            sh '''
                                export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                                export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                                terraform apply -auto-approve
                            '''
                    }
                }
            }
        }

        stage('Run Ansible') {
            steps {
                dir('ansible') {
                    withCredentials([sshUserPrivateKey(credentialsId: 'ec2-key',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER')]) {
                        sh '''
                            # جلب ال IP من Terraform output
                            EC2_IP=$(terraform -chdir=../terraform output -raw public_ip)

                            # إنشاء ملف inventory ديناميكي
                            echo "$EC2_IP ansible_user=$SSH_USER ansible_ssh_private_key_file=$SSH_KEY" > inventory

                            # تشغيل الـ playbook باستخدام الـ inventory
                            ansible-playbook -i inventory playbook.yml
                        '''
                    }
                }
            }
        }
    }
}
