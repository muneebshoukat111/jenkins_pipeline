pipeline {
    agent any

    environment {
        // Docker & Kubernetes details
        IMAGE_NAME              = "muneebshoukat/test"
        IMAGE_TAG               = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID   = "e0185fe0-af38-4847-9e87-bed5e756348f"
        KUBERNETES_CREDENTIALS_ID = "000"       // Jenkins credential ID for kubeconfig
        K8S_NAMESPACE    // K8S_SERVER_URL          = "https:        = "webapps"
       //192.168.0.173:16443"
        
        // Set the KUBECONFIG environment variable from Jenkins credentials
        KUBECONFIG = credentials("${KUBERNETES_CREDENTIALS_ID}")
    }

    stages {
        stage('Install Helm') {
            steps {
                sh '''
                    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
                    chmod 700 get_helm.sh
                    ./get_helm.sh
                '''
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the versioned Docker image
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
                    // Also tag the image as "latest"
                    docker.build("${IMAGE_NAME}:latest", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push both the versioned and the latest images
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
                    // Remove local images to free up space on the Jenkins agent
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
            cleanWs()
        }
    }
}
