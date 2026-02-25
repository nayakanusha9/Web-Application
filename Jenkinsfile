pipeline {
    agent any 

    parameters {
        string(name: 'VERSION', description: 'Enter the APP Version')
    }

    environment {
        AWS_ACCOUNT_ID = "372123585601"
        REGION = "us-east-1"
        REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/devops"
        DOCKER_IMAGE = "turbo-grocery-app:${VERSION}"
        DOCKER_REGISTRY = "docker.io"
        DOCKER_REGISTRY_CREDENCIALS = "docker_creds"
        CONTAINER_NAME = "turbo-grocery-container"
        APP_PORT = "5000"
    }

    stages {
        stage('Clone') {
            steps {
                echo "Cloing the GitHub Repository"
                git url: 'https://github.com/nayakanusha9/Turbo-Grocery.git', branch: 'main'
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building the Docker Image ${DOCKER_IMAGE}"
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    withAWS(credentials: 'aws_creds', region: "${REGION}") {
                        echo "Pushing the docker image to AWS ECR"
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${REPO_URI}
                            docker tag ${DOCKER_IMAGE} ${REPO_URI}:${VERSION}
                            docker push ${REPO_URI}:${VERSION}
                        """
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDENCIALS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                  echo "Pushing the docker image to Docker Hub"
                  sh """
                    docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD}
                    docker tag ${DOCKER_IMAGE} anushanayak091/${DOCKER_IMAGE}
                    docker push anushanayak091/${DOCKER_IMAGE}
                    """  
            }
            }
        }
        stage('Run Container') {
            steps {
                echo "Running Docker Container on Jenkins EC2 instance"
                sh """
                    docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:80 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        success {
            echo " Build & Deployment Successful! Access the app at: http://<EC2-PUBLIC-IP>:${APP_PORT}"
        }
        failure {
            echo " Build Failed. Check logs."
        }
    }
}
