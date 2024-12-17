pipeline {
    agent any

    environment {
        // Docker & Kubernetes details (if needed)
        IMAGE_NAME              = "muneebshoukat/test"
        IMAGE_TAG               = "0.1.${BUILD_NUMBER}"
        DOCKER_CREDENTIALS_ID   = "e0185fe0-af38-4847-9e87-bed5e756348f"
        KUBERNETES_CREDENTIALS_ID = "000" // Jenkins credential ID for your Kubeconfig
        K8S_NAMESPACE           = "webapps" // or whichever namespace you want to deploy into
        
        // Set KUBECONFIG from Jenkins credentials
        KUBECONFIG = credentials("${KUBERNETES_CREDENTIALS_ID}")
    }

    stages {
        stage('Install Helm') {
            steps {
                sh '''
                    #!/bin/bash
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

        // Optional stages if you want to build and push Docker images
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

        // The new deployment stage
        stage('Deploy Helm Chart') {
            steps {
                script {
                    // Change directory to where your Helm chart is located
                    sh '''
                        cd /jenkins_pipeline/chart
                        # For a fresh install or an upgrade
                        helm upgrade --install muneeb . \
                            --namespace ${K8S_NAMESPACE} \
                            --create-namespace \
                            -f values.yaml
                    '''
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
