// pipeline {
//     agent any

//     environment {
//         IMAGE_NAME = "muneebshoukat/test"
//         IMAGE_TAG = "0.1.${BUILD_NUMBER}"
//         DOCKER_CREDENTIALS_ID = "e0185fe0-af38-4847-9e87-bed5e756348f"
//         KUBERNETES_CREDENTIALS_ID = "k8s.connect"
//         K8S_NAMESPACE = "webapps"
//         K8S_SERVER_URL = "https://192.168.0.173:16443"
//     }

//     stages {
//         stage('Install Helm') {
//             steps {
//                 sh '''
//                 curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
//                 chmod 700 get_helm.sh
//                 ./get_helm.sh
//                 '''
//             }
//         }

//         stage('Clone Repository') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/muneebshoukat111/jenkins_pipeline.git'
//             }
//         }

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

//         stage('K8-Deploy') {
//             steps {
//                 withKubeConfig(
//                     caCertificate: '', // Optional
//                     clusterName: 'microk8s-cluster',
//                     contextName: '', // Optional
//                     credentialsId: "${KUBERNETES_CREDENTIALS_ID}",
//                     namespace: "${K8S_NAMESPACE}",
//                     restrictKubeConfigAccess: false,
//                     serverUrl: "${K8S_SERVER_URL}"
//                 ) {
//                     script {
//                         sh '''
//                         kubectl config view
//                         kubectl config current-context
//                         kubectl get pods --namespace=${K8S_NAMESPACE}
//                         kubectl get svc --namespace=${K8S_NAMESPACE}
//                         '''
//                     }
//                 }
//             }
//         }
//     }

//     post {
//         always {
//             cleanWs()
//         }
//     }
// }
pipeline {
    agent any
    stages {
        stage('Check Pods in Jenkins Namespace') {
            steps {
                withKubeCredentials(kubectlCredentials: [[
                    caCertificate: '',           // Add CA data if needed
                    clusterName: 'minikube',     // Your cluster name
                    contextName: 'jenkins-ctx',  // A name for the context (optional)
                    credentialsId: 'test',       // The ID of your Kubernetes credentials in Jenkins
                    namespace: 'jenkins',        // The namespace you want to check
                    serverUrl: 'https://192.168.49.2:8443' // Your cluster's API server URL
                ]]) {
                    sh 'kubectl get pods -n jenkins'
                }
            }
        }
    }
}
