pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

stage('Ansible Syntax Check') {
    steps {
        withCredentials([
            string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
            string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
        ]) {
            sh '''
            
                ansible-playbook \
                --syntax-check \
                -i inventory/aws_ec2.yml \
                playbook.yml
            '''
        }
    }
}

        stage('Continuous Deployment') {
            steps {
                sshagent(credentials: ['ansible-ssh-key']) {
                    sh '''
                        ansible-playbook \
                        -i inventory/aws_ec2.yml \
                        playbook.yml
                    '''
                }
            }
        }

        stage('Deployment Validation') {
            steps {
                sshagent(credentials: ['ansible-ssh-key']) {
                    sh '''
                        ansible \
                        -i inventory/aws_ec2.yml \
                        webservers \
                        -m shell \
                        -a "systemctl is-active nginx"

                        ansible \
                        -i inventory/aws_ec2.yml \
                        webservers \
                        -m shell \
                        -a "curl -s http://localhost"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Ansible CD deployment completed successfully.'
        }

        failure {
            echo 'Ansible CD deployment failed.'
        }
    }
}
