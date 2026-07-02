pipeline {
    agent any

    environment {
        IMAGE_NAME = "vivinmathew/portfolio"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_NAME = "portfolio"
        APP_PORT = "8000"
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
                    docker run -d --name portfolio-test -p ${APP_PORT}:${APP_PORT} ${IMAGE_NAME}:${IMAGE_TAG}
                    sleep 3
                    curl -sf http://localhost:${APP_PORT}/ || (docker logs portfolio-test && exit 1)
                    docker stop portfolio-test && docker rm portfolio-test
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} || true
                    docker rm ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:${APP_PORT} ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline passed - ${IMAGE_NAME}:${IMAGE_TAG} deployed on port ${APP_PORT}"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
