pipeline {
    agent any

    environment {
        IMAGE_NAME = "muneebshoukat/test"
        IMAGE_TAG = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
        NAMESPACE = 'wabapps'
        HELM_CHART_PATH = './chart/muneeb'
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

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig(
                    caCertificate: '', 
                    clusterName: 'microk8s-cluster', 
                    contextName: '', 
                    credentialsId: 'k8s.connect', 
                    namespace: 'wabapps', 
                    restrictKubeConfigAccess: false, 
                    serverUrl: 'https://127.0.0.1:16443'
                ) {
                    script {
                        // Create namespace if not exists
                        sh """
                        kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f - || true
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
    }

    post {
        always {
            cleanWs()
        }
    }
}
