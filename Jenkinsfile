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
                sh '''
                    ansible-playbook \
                    --syntax-check \
                    -i inventory/hosts.ini \
                    playbook.yml
                '''
            }
        }

        stage('Continuous Deployment') {
            steps {
                sshagent(credentials: ['ansible-ssh-key']) {
                    sh '''
                        ansible-playbook \
                        -i inventory/hosts.ini \
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
                        -i inventory/hosts.ini \
                        webservers \
                        -m shell \
                        -a "systemctl is-active nginx"

                        ansible \
                        -i inventory/hosts.ini \
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
