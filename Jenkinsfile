pipeline {
    agent any
    environment {
        REMOTE_HOST = 'useradmin@13.95.14.175'
        SSH_CREDENTIALS_ID = 'AppServer' // SSH credentials for remote access
        DOCKER_IMAGE = 'express'
        DOCKER_TAG = 'latest'
        AZURE_USER = 'useradmin'
        AZURE_VM_IP = '13.95.14.175'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker pull ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
        stage('Save Docker Image') {
            steps {
                sh """
                docker save -o ${DOCKER_IMAGE}.tar ${DOCKER_IMAGE}:${DOCKER_TAG}
                """
            }
        }
        stage('Copy to Azure VM') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    sh """
                    scp ${DOCKER_IMAGE}.tar ${AZURE_USER}@${AZURE_VM_IP}:/tmp/
                    """
                }
            }
        }
        stage('Deploy to Azure VM') {
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS_ID}"]) {
                    script {
                        try {
                            sh """
                            ssh ${AZURE_USER}@${AZURE_VM_IP} << 'EOF'
                            docker load < /tmp/${DOCKER_IMAGE}.tar
                            docker run -d --name express-app-container -p 80:3000 ${DOCKER_IMAGE}:${DOCKER_TAG}
                            EOF
                            """
                        } catch (Exception e) {
                            echo "Ignoring non-critical error: ${e.getMessage()}"
                        }
                    }
                }
            }
        }
    }
}
