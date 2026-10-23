
pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'vishwanath57'
        IMAGE_NAME = 'cicd-webapp'
        EC2_HOST = '15.252.236.50'
        EC2_USER = 'ubuntu'
        CONTAINER_NAME = 'cicd-webapp'
        APP_PORT = '5000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m venv .venv-ci
                    . .venv-ci/bin/activate
                    python -m pip install --upgrade pip
                    python -m pip install -r requirements.txt
                '''
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh '''
                    . .venv-ci/bin/activate
                    python -m unittest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} \
                      -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKERHUB_TOKEN" | docker login \
                          --username "$DOCKERHUB_USER" \
                          --password-stdin

                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(credentials: ['ec2-deploy-key']) {
                    sh '''
                        set -eu

                        echo "Connecting to EC2 and deploying application..."

                        ssh -o BatchMode=yes \
                            -o ConnectTimeout=15 \
                            ${EC2_USER}@${EC2_HOST} \
                            "sudo docker pull ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} &&
                             (sudo docker stop ${CONTAINER_NAME} || true) &&
                             (sudo docker rm ${CONTAINER_NAME} || true) &&
                             sudo docker run -d \
                               --name ${CONTAINER_NAME} \
                               --restart unless-stopped \
                               -p ${APP_PORT}:5000 \
                               ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${BUILD_NUMBER} &&
                             sleep 3 &&
                             sudo docker inspect -f '{{.State.Running}}' ${CONTAINER_NAME} | grep -qx true"

                        echo "Deployment completed successfully."
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline succeeded. Application deployed to EC2!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output for details.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}

