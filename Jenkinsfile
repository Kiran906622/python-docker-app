pipeline {
    agent any

    environment {
        APP_EC2 = '13.201.227.106'         // Your RHEL App Server IP
        SSH_CREDENTIALS = 'app-server-ssh' // Your Jenkins Credential ID
        APP_DIR = '/home/ec2-user/myapp'    // RHEL home directory
        DOCKER_IMAGE = 'myapp:latest'
        GIT_REPO = 'https://github.com/Kiran906622/python-docker-app.git'
        GIT_BRANCH = 'dev'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Pulls code to the Jenkins server
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: 'github-creds' 
            }
        }

        stage('Deploy & Build on Remote RHEL') {
            steps {
                sshagent(["${SSH_CREDENTIALS}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ec2-user@${APP_EC2} '
                        echo "1. Clearing Port 5000 (Killing ghost processes)..."
                        # This solves the "Address already in use" error
                        sudo fuser -k 5000/tcp || true
                        
                        echo "2. Cleaning up old directory..."
                        sudo rm -rf ${APP_DIR}
                        
                        echo "3. Cloning fresh code from GitHub..."
                        git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR}
                        
                        cd ${APP_DIR}
                        
                        echo "4. Removing old Docker container if exists..."
                        sudo docker stop myapp || true
                        sudo docker rm myapp || true
                        
                        echo "5. Building new Docker image..."
                        sudo docker build -t ${DOCKER_IMAGE} .
                        
                        echo "6. Starting new container on port 5000..."
                        sudo docker run -d -p 5000:5000 --name myapp ${DOCKER_IMAGE}
                        
                        echo "7. Verifying container status..."
                        sudo docker ps | grep myapp
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "-----------------------------------------------------------"
            echo "✅ DEPLOYMENT SUCCESSFUL"
            echo "URL: http://${APP_EC2}:5000"
            echo "-----------------------------------------------------------"
        }
        failure {
            echo "-----------------------------------------------------------"
            echo "❌ DEPLOYMENT FAILED"
            echo "Check Console Output for the specific error."
            echo "-----------------------------------------------------------"
        }
    }
}
