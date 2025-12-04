pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('Dockerhub_pwd')
        IMAGE_NAME = "imem11/validation04/12/2025"
        IMAGE_TAG = "${sh(script:'git rev-parse --short HEAD', returnStdout: true).trim()}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn -version"
                sh "mvn clean package -DskipTests"    // petclinic uses "package" not "install"
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                    echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                    docker logout
                """
            }
        }
    }

    post {
        always {
            deleteDir()
            sh 'docker system prune -f || true'
        }
        success { echo "Image pushed → https://hub.docker.com/r/${IMAGE_NAME}" }
    }
}