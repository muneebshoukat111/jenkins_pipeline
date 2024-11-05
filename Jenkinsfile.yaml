pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "muneebshoukat/test" // DockerHub repository name
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f" // DockerHub credentials ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'test', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
            }
        }

        stage('Get and Increment Version') {
            steps {
                script {
                    // Read version from version.txt
                    def versionFile = readFile('version.txt').trim()
                    def versionParts = versionFile.tokenize('.').collect { it as int }
                    versionParts[-1] = versionParts[-1] + 1  // Increment the patch version
                    def newVersion = versionParts.join('.')
                    
                    // Set new version in environment variable for use in later stages
                    env.IMAGE_TAG = newVersion

                    // Write updated version back to version.txt
                    writeFile file: 'version.txt', text: newVersion
                    echo "New version is: ${newVersion}"
                }
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

        stage('Commit and Push Version Update') {
            steps {
                script {
                    sh 'git config user.email "jenkins@example.com"'
                    sh 'git config user.name "Jenkins"'
                    sh 'git add version.txt'
                    sh 'git commit -m "Bump version to ${IMAGE_TAG}"'
                    sh 'git push origin test'
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
