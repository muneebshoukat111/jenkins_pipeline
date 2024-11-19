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

    environment {
        K8S_NAMESPACE = "webapps"
        K8S_SERVER_URL = "https://192.168.0.173:16443" // Kubernetes API server URL
        K8S_USER_TOKEN = credentials('k8s-user-token-id') // Replace with Jenkins credentials ID
    }

    stages {
        stage('Setup Kubernetes Config') {
            steps {
                script {
                    // Create a kubeconfig file with the token from Jenkins credentials
                    writeFile file: 'kubeconfig', text: """
                    apiVersion: v1
                    kind: Config
                    clusters:
                    - cluster:
                        server: ${K8S_SERVER_URL}
                        insecure-skip-tls-verify: true
                      name: hardcoded-cluster
                    contexts:
                    - context:
                        cluster: hardcoded-cluster
                        namespace: ${K8S_NAMESPACE}
                        user: hardcoded-user
                      name: hardcoded-context
                    current-context: hardcoded-context
                    users:
                    - name: hardcoded-user
                      user:
                        token: ${K8S_USER_TOKEN}
                    """
                }
            }
        }

        stage('Ensure Namespace Exists') {
            steps {
                script {
                    sh '''
                    export KUBECONFIG=kubeconfig
                    kubectl create namespace ${K8S_NAMESPACE} || echo "Namespace already exists"
                    '''
                }
            }
        }

        stage('Test Kubernetes Connection') {
            steps {
                script {
                    // Test Kubernetes access
                    sh '''
                    export KUBECONFIG=kubeconfig
                    kubectl get pods --namespace=${K8S_NAMESPACE}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Create a deployment, verify it, and delete it
                    sh '''
                    export KUBECONFIG=kubeconfig
                    kubectl create deployment test-deployment --image=nginx --namespace=${K8S_NAMESPACE}
                    kubectl get deployments --namespace=${K8S_NAMESPACE}
                    kubectl delete deployment test-deployment --namespace=${K8S_NAMESPACE}
                    '''
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace to remove kubeconfig and other temporary files
            cleanWs()
        }
    }
}
