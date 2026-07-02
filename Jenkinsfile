pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "vivinmathew/portfolio"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "portfolio"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('Test') {
            steps {
                sh '''
                    docker run -d --name portfolio-test -p 8080:80 ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 3
                    curl -sf http://localhost:8080/ || (docker logs portfolio-test && exit 1)
                    docker stop portfolio-test && docker rm portfolio-test
                '''
            }
        }

        stage('Push') {
            steps {
                sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 80:80 ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline passed - ${IMAGE_NAME}:${IMAGE_TAG} deployed"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
