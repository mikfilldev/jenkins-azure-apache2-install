pipeline {
    agent any
    
    environment {
        CREDENTIALS_ID = '7baefb58-ecaf-42c8-b123-88f1fb4deb3a'
        APACHE2_ACCESS_LOG_FILE = '/var/log/apache2/access.log'
        APACHE2_ERROR_LOG_FILE = '/var/log/apache2/error.log'
    }
    stages {
        stage('TestSSH connection to Azure Instance') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AZURE_INSTANCE_IP', variable: 'INSTANCE_IP'),
                                    string(credentialsId: 'AZURE_SSH_USER', variable: 'SSH_USER')]) {
                        sshagent(credentials: [env.CREDENTIALS_ID]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.INSTANCE_IP}
                            """
                        }
                    }                        
                }
            }
        }
        stage('Get machine overview') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AZURE_INSTANCE_IP', variable: 'INSTANCE_IP'),
                                    string(credentialsId: 'AZURE_SSH_USER', variable: 'SSH_USER')]) {
                        sshagent(credentials: [env.CREDENTIALS_ID]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.INSTANCE_IP} << 'EOF'
                                uname -a
                                df -h
                                curl ifconfig.me
                            """
                        }
                    }                        
                }
            }
        }
        stage('Update packages') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AZURE_INSTANCE_IP', variable: 'INSTANCE_IP'),
                                    string(credentialsId: 'AZURE_SSH_USER', variable: 'SSH_USER')]) {
                        sshagent(credentials: [env.CREDENTIALS_ID]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.INSTANCE_IP} << 'EOF'
                                sudo apt update
                                sudo apt upgrade -y
                            """
                        }
                    }                        
                }
            }
        }
        stage('Install apache2') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AZURE_INSTANCE_IP', variable: 'INSTANCE_IP'),
                                    string(credentialsId: 'AZURE_SSH_USER', variable: 'SSH_USER')]) {
                        sshagent(credentials: [env.CREDENTIALS_ID]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.INSTANCE_IP} << 'EOF'
                                sudo apt install apache2 -y
                            """
                        }
                    }                        
                }
            }
        }
        stage('Check Apache2 Logs for 4xx and 5xx Errors') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AZURE_INSTANCE_IP', variable: 'INSTANCE_IP'),
                                    string(credentialsId: 'AZURE_SSH_USER', variable: 'SSH_USER')]) {
                        sshagent(credentials: [env.CREDENTIALS_ID]) {
                            sh """
                                ssh -o StrictHostKeyChecking=no ${env.SSH_USER}@${env.INSTANCE_IP} << 'EOF'
                                echo "Checking Apache2 access log for 4xx and 5xx errors..."
                                grep -E 'HTTP/[0-9\\.]+\" (4[0-9][0-9]|5[0-9][0-9])' ${APACHE2_ACCESS_LOG_FILE} > /tmp/access_error_log.txt
                                if [ -s /tmp/access_error_log.txt ]; then
                                    echo "Errors found in the access log:"
                                    cat /tmp/access_error_log.txt
                                else
                                    echo "No 4xx or 5xx errors found in the access log."
                                fi
                                echo "Checking Apache2 error log for errors..."
                                grep -E '.*' ${APACHE2_ERROR_LOG_FILE} > /tmp/error_log.txt
                                if [ -s /tmp/error_log.txt ]; then
                                    echo "Errors found in the error log:"
                                    cat /tmp/error_log.txt
                                else
                                    echo "No errors found in the error log."
                                fi
                            """
                        }
                    }                        
                }
            }
        }
    }
}