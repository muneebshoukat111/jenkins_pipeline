pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}" // Incremented tag with each build
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
    }

    post {
        always {
            // Clean up the workspace
            deleteDir()
        }
    }
}
