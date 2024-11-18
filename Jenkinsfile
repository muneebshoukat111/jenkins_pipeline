pipeline {
    agent any

    environment {
        IMAGE_NAME = "muneebshoukat/test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
        KUBERNETES_CREDENTIALS_ID = "k8s.connect"
        K8S_NAMESPACE = "webapps"
        K8S_SERVER_URL = "https://127.0.0.1:16443"
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
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
                    sh "docker rmi ${IMAGE_NAME}:latest || true"
                }
            }
        }

        stage('K8-Deploy') {
            steps {
                withKubeConfig(
                    caCertificate: '', // Optional, leave empty unless you need a CA certificate
                    clusterName: 'microk8s-cluster', // Descriptive cluster name
                    contextName: '', // Optional, only use if you need a specific kube-context
                    credentialsId: "${KUBERNETES_CREDENTIALS_ID}", // Jenkins credentials ID for Kubernetes
                    namespace: "${K8S_NAMESPACE}", // Target namespace
                    restrictKubeConfigAccess: false, // Default behavior
                    serverUrl: "${K8S_SERVER_URL}" // Kubernetes API server URL
                ) {
                    sh '''
                    kubectl apply -f deployment-service.yml
                    kubectl get pods
                    kubectl get svc
                    '''
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
