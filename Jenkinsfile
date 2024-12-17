// pipeline {
//     agent any

//     environment {
//         IMAGE_NAME            = "muneebshoukat/test"
//         IMAGE_TAG             = "0.1.${BUILD_NUMBER}"
//         DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
//         K8S_NAMESPACE         = "muneeb"
//         KUBECONFIG            = credentials('000')  // Jenkins credential ID for kubeconfig
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
//                 script {
//                     sh '''
//                         # Add your desired Helm repo. For example, the official NGINX stable repo:
//                         helm repo add nginx-stable https://helm.nginx.com/stable
//                         helm repo update

//                         # Install a chart named "nginx-ingress" from the added repo
//                         helm install my-nginx nginx-stable/nginx-ingress \
//                             --namespace ${K8S_NAMESPACE} \
//                             --create-namespace
//                     '''
//                 }
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

    environment {
       
        KUBECONFIG = credentials('000')
    }

    stages {
        stage('Create Namespace') {
            steps {
                sh 'kubectl create namespace muneeb-finale'
                // sh 'kubectl  get pod -n muneeb-finale'
                sh 'kubectl get  pod -n muneeb'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}