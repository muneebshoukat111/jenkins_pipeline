pipeline {
    agent any

    environment {
        IMAGE_NAME = "muneebshoukat/test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
        NAMESPACE = 'muneeb'
        HELM_CHART_PATH = './chart/muneeb'
        KUBECONFIG = "/home/muneeb/.kube/config" 
        K8S_TOKEN = "2542492c1f7a2ce3d12cea14f2db96c19901957c02d894571a4efd3bfcd2a253" 
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

        stage('Deploy to Local Kubernetes') {
            steps {
                script {
                    // Set Kubernetes context
                    sh "kubectl config set-context jenkins-context --cluster=<cluster-name> --namespace=${NAMESPACE} --user=jenkins-user"
                    sh "kubectl config use-context jenkins-context"

                    // Create namespace if not exists, with validation disabled
                    sh """
                    kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f - --validate=false || true
                    """

                    // Deploy Helm chart
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
            cleanWs()
        }
    }
}
