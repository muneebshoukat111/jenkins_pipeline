pipeline {
    agent any

    environment {
        IMAGE_NAME = "muneebshoukat/test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
        NAMESPACE = 'muneeb' // Target namespace in Kubernetes
        HELM_CHART_PATH = './muneeb' // Path to your Helm chart
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
                    docker.build("${IMAGE_NAME}:latest", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:latest").push()
                    }
                }
            }
        }

        stage('Cleanup Docker Image') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to Local Kubernetes') {
            steps {
                script {
                    // Ensure namespace exists
                    sh "kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"

                    // Deploy the Helm chart with the image tag as a value override
                    sh """
                        helm upgrade --install my-release ${HELM_CHART_PATH} \
                            --namespace ${NAMESPACE} \
                            --set image.repository=${IMAGE_NAME} \
                            --set image.tag=${IMAGE_TAG}
                    """
                }
            }
        }
    }

    post {
        always {
            deleteDir()
        }
    }
}
