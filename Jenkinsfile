// pipeline {
//     agent any

//     triggers {
//         githubPush()  // This will trigger the pipeline when a new commit is pushed to GitHub
//     }

//     environment {
//         IMAGE_NAME            = "muneebshoukat/test"
//         IMAGE_TAG             = "0.1.${BUILD_NUMBER}"
//         DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
//         K8S_NAMESPACE         = "muneeb-finale"
//         KUBECONFIG            = credentials('1234')  // Jenkins credential ID for kubeconfig
//     }

//     stages {
//         stage('Install Helm') {
//             steps {
//                 sh '''
//                     #!/bin/bash
//                     curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
//                     chmod 700 get_helm.sh
//                     ./get_helm.sh
//                 '''
//             }
//         }

//         stage('Clone Repository') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
//             }
//         }

//         // Optional: Build & push Docker image if needed
//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
//                     docker.build("${IMAGE_NAME}:latest", ".")
//                 }
//             }
//         }

//         stage('Push Docker Image') {
//             steps {
//                 script {
//                     docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
//                         docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
//                         docker.image("${IMAGE_NAME}:latest").push()
//                     }
//                 }
//             }
//         }

//         stage('Cleanup Docker Image') {
//             steps {
//                 script {
//                     sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
//                     sh "docker rmi ${IMAGE_NAME}:latest || true"
//                 }
//             }
//         }

//         stage('Deploy Helm Chart') {
//             steps {
//                 sh """
//                     helm upgrade my-release ./infra/app \
//                         --namespace ${K8S_NAMESPACE} \
//                         --set image.repository=${IMAGE_NAME} \
//                         --set image.tag=latest 
//                 """
//             }
//         }
//     }

//     post {
//         always {
//             echo "Pipeline completed."
//             cleanWs()
//         }
//     }
// }
pipeline {
    agent any

    triggers {
        githubPush()  // Triggers the pipeline when a new commit is pushed to GitHub
    }

    environment {
        IMAGE_NAME    = "muneebshoukat/test"
        IMAGE_TAG     = "latest"  // Use the desired tag from DockerHub
        K8S_NAMESPACE = "muneeb-f"
        KUBECONFIG    = credentials('1234')  // Jenkins credential ID for kubeconfig
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

        stage('Create Namespace') {
            steps {
                sh """
                    kubectl create namespace ${K8S_NAMESPACE} || echo "Namespace ${K8S_NAMESPACE} already exists."
                """
            }
        }

        stage('Apply DockerHub Secret') {
            steps {
                withCredentials([file(credentialsId: 'dockerhub-secret-yaml', variable: 'DOCKER_SECRET_YAML')]) {
                    sh """
                        kubectl apply -f \$DOCKER_SECRET_YAML
                    """
                }
            }
        }

        stage('Deploy Helm Chart') {
            steps {
                sh """
                    helm upgrade --install my-release ./infra/app \
                        --namespace ${K8S_NAMESPACE} \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG}
                """
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
            cleanWs()
        }
        failure {
            echo "Pipeline failed. Check the logs for more details."
            // Optionally, add notifications (e.g., email, Slack) here
        }
    }
}

