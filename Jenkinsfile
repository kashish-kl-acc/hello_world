pipeline {
    agent any

    environment {
        IMAGE_NAME = "kashish2001/hello-world"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        DOCKER_CREDS = credentials('dockerhub-credentials')
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Login to Docker Hub') {
            steps {
                sh """
                echo ${DOCKER_CREDS_PSW} | docker login \
                  -u ${DOCKER_CREDS_USR} \
                  --password-stdin
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                sh """
                docker push ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Update Deployment Manifest') {
            steps {
                sh """
                sed -i 's|image: .*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' deployment.yaml
                """
            }
        }

        stage('Commit & Push Manifest') {
            steps {
                sh """
                git config user.name "jenkins"
                git config user.email "jenkins@ci.local"

                git add deployment.yaml
                git commit -m "Update image to ${IMAGE_NAME}:${IMAGE_TAG}"
                git push
                """
            }
        }
    }

    post {
        success {
            echo "Docker image pushed and deployment.yaml updated successfully"
        }
    }
}
