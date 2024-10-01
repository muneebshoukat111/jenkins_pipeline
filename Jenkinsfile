pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                script {
                    echo 'Hello, Jenkins!'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    echo 'Building Docker image...'
                    sh 'docker build -t your-image-name:latest .'
                }
            }
        }
    }
}
