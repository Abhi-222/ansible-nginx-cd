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
                        export AWS_DEFAULT_REGION=ap-south-1

                        ansible-playbook \
                        --syntax-check \
                        -i inventory/aws_ec2.yml \
                        -e ansible_user=ubuntu \
                        playbook.yml
                    '''
                }
            }
        }

        stage('Test SSH Connection') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sshagent(credentials: ['ansible-ssh-key']) {
                        sh '''
                            export AWS_DEFAULT_REGION=ap-south-1

                            echo "Loaded SSH keys:"
                            ssh-add -L

                            echo "Testing Ansible SSH connection..."

                            ansible \
                            -i inventory/aws_ec2.yml \
                            -e ansible_user=ubuntu \
                            webserver \
                            -m ping
                        '''
                    }
                }
            }
        }

        stage('Continuous Deployment') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sshagent(credentials: ['ansible-ssh-key']) {
                        sh '''
                            export AWS_DEFAULT_REGION=ap-south-1

                            ansible-playbook \
                            -i inventory/aws_ec2.yml \
                            -e ansible_user=ubuntu \
                            playbook.yml
                        '''
                    }
                }
            }
        }

        stage('Deployment Validation') {
            steps {
                withCredentials([
                    string(credentialsId: 'aws-access-key', variable: 'AWS_ACCESS_KEY_ID'),
                    string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_ACCESS_KEY')
                ]) {
                    sshagent(credentials: ['ansible-ssh-key']) {
                        sh '''
                            export AWS_DEFAULT_REGION=ap-south-1

                            echo "Checking Nginx service..."

                            ansible \
                            -i inventory/aws_ec2.yml \
                            -e ansible_user=ubuntu \
                            webserver \
                            -m shell \
                            -a "systemctl is-active nginx"

                            echo "Checking deployed webpage..."

                            ansible \
                            -i inventory/aws_ec2.yml \
                            -e ansible_user=ubuntu \
                            webserver \
                            -m shell \
                            -a "curl -s http://localhost"
                        '''
                    }
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
