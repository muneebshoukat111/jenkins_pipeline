pipeline {
    agent any

    environment {
        IMAGE_NAME = "muneebshoukat/test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
        NAMESPACE = 'muneeb'
        HELM_CHART_PATH = './chart/muneeb'
        KUBECONFIG = "/home/muneeb/.kube/config" // Ensure Minikube KUBECONFIG path is set
        K8S_TOKEN = "2542492c1f7a2ce3d12cea14f2db96c19901957c02d894571a4efd3bfcd2a253" // Hard-coded Kubernetes token
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image with both a specific tag and the "latest" tag
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
                    docker.build("${IMAGE_NAME}:latest", ".")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    // Push Docker images to Docker Hub with provided credentials
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
                    // Remove Docker images from the local workspace to free up space
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        stage('Deploy to Local Kubernetes') {
            steps {
                script {
                    // Configure kubectl with the hard-coded token for authentication
                    sh "kubectl config set-credentials jenkins-user --token=${K8S_TOKEN}"

                    // Ensure namespace exists, using --dry-run to prevent errors if it already exists
                    sh """
                    kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f - || true
                    """

                    // Deploy the Helm chart, specifying the image repository and tag
                    sh """
                    helm upgrade --install test ${HELM_CHART_PATH} \
                        --namespace ${NAMESPACE} \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG} --debug
                    """
                }
            }
        }
    }

    post {
        always {
            // Clean workspace to remove files and free up space
            cleanWs() 
            // Alternative cleanup if cleanWs fails
            // deleteDir()
        }
    }
}
