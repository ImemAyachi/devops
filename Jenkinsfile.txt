pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('Dockerhub_pwd')
        IMAGE_NAME = "imem11/test1"
        // Dynamic tag using short Git commit hash + "latest"
        IMAGE_TAG = "${sh(script:'git rev-parse --short HEAD', returnStdout: true).trim()}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', 
                    url: 'https://github.com/ImemAyachi/devops.git'
            }
        }

        stage('Maven Verify') {
            steps {
                sh "mvn -version"
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                """
            }
        }

        stage('Docker Push') {
            steps {
                sh 'echo $DOCKER_HUB_CREDENTIALS_PSW | docker login -u $DOCKER_HUB_CREDENTIALS_USR --password-stdin'
                sh """
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    docker push ${IMAGE_NAME}:latest
                """
                // Optional: logout for security
                sh 'docker logout'
            }
        }
    }

    post {
        always {
            // Clean workspace + remove dangling images
            cleanWs()
            sh 'docker system prune -f || true'
        }
        success {
            echo "Image successfully pushed to Docker Hub: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}