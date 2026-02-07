pipeline {
    agent any

    environment {
        APP_EC2 = '13.201.227.106'         
        SSH_CREDENTIALS = 'app-server-ssh' 
        APP_DIR = '/home/ec2-user/myapp'    // RHEL default home
        DOCKER_IMAGE = 'myapp:latest'
        GIT_REPO = 'https://github.com/Kiran906622/python-docker-app.git'
        GIT_BRANCH = 'dev'
    }

    stages {
        stage('Checkout Code') {
            steps {
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
                        echo "Cleaning up old directory..."
                        sudo rm -rf ${APP_DIR}
                        
                        echo "Cloning code..."
                        git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR}
                        
                        cd ${APP_DIR}
                        
                        echo "Stopping old container..."
                        sudo docker stop myapp || true
                        sudo docker rm myapp || true
                        
                        echo "Building new image..."
                        sudo docker build -t ${DOCKER_IMAGE} .
                        
                        echo "Starting container..."
                        sudo docker run -d -p 5000:5000 --name myapp ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }
}
