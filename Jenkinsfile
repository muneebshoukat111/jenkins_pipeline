pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "muneebshoukat/test" // DockerHub repository name
        IMAGE_TAG = "0.1.${BUILD_NUMBER}" // Incremented tag with each build
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f" // DockerHub credentials ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'test', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Login to DockerHub using Jenkins credentials
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        // Push the Docker image
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Cleanup Docker Image') {
            steps {
                script {
                    // Remove the local Docker image after pushing to DockerHub
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }
    }

    post {
        always {
            // Clean up the workspace
            deleteDir()
        }
    }
}