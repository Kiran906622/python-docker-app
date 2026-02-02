pipeline {
    agent any

    environment {
        APP_EC2 = '43.204.112.114'        // App EC2 IP
        SSH_CREDENTIALS = 'app-ec2-ssh'   // Jenkins SSH credential ID
        APP_DIR = '/home/redhat/myapp'    // Folder on App EC2
        DOCKER_IMAGE = 'myapp:latest'
        GIT_REPO = 'https://github.com/Kiran906622/python-docker-app.git'
        GIT_BRANCH = 'dev'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: "${GIT_BRANCH}",
                    url: "${GIT_REPO}",
                    credentialsId: "${SSH_CREDENTIALS}"
            }
        }

        stage('Deploy Docker App on Remote EC2') {
            steps {
                sshagent(['app-ec2-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no redhat@${APP_EC2} '
                        echo "Stopping old container..."
                        docker stop myapp || true
                        docker rm myapp || true
                        mkdir -p ${APP_DIR}
                        cd ${APP_DIR}
                        echo "Pulling latest code..."
                        git pull origin ${GIT_BRANCH} || git clone -b ${GIT_BRANCH} ${GIT_REPO} ${APP_DIR}
                        echo "Building Docker image..."
                        docker build -t ${DOCKER_IMAGE} .
                        echo "Running new container..."
                        docker run -d -p 5000:5000 --name myapp ${DOCKER_IMAGE}
                    '
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Su  ccessful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}

