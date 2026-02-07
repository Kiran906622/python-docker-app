pipeline {
    agent any

    environment {
        APP_EC2 = '13.201.227.106'         // Your updated App EC2 IP
        SSH_CREDENTIALS = 'app-server-ssh' // The ID you created in Jenkins
        APP_DIR = '/home/ubuntu/myapp'    
        DOCKER_IMAGE = 'myapp:latest'
        GIT_REPO = 'https://github.com/Kiran906622/python-docker-app.git'
        GIT_BRANCH = 'dev'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // This pulls the code to Jenkins so it can read this script
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: 'github-creds' 
            }
        }

        stage('Deploy & Build on Remote EC2') {
            steps {
                // Tells Jenkins to use the 'app-server-ssh' key
                sshagent(["${SSH_CREDENTIALS}"]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no ubuntu@${APP_EC2} '
                        echo "Cleaning up old directory..."
                        sudo rm -rf ${APP_DIR}
                        
                        echo "Cloning code to App Server..."
                        git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR}
                        
                        cd ${APP_DIR}
                        
                        echo "Stopping existing container..."
                        sudo docker stop myapp || true
                        sudo docker rm myapp || true
                        
                        echo "Building new Docker image..."
                        sudo docker build -t ${DOCKER_IMAGE} .
                        
                        echo "Starting container on port 5000..."
                        sudo docker run -d -p 5000:5000 --name myapp ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Successfully deployed to http://${APP_EC2}:5000"
        }
        failure {
            echo "❌ Deployment failed. Check the Console Output."
        }
    }
}
