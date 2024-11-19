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
        K8S_SERVER_URL = "https://127.0.0.1:16443" // Kubernetes API server URL
        K8S_USER_TOKEN = "eyJhbGciOiJSUzI1NiIsImtpZCI6IndrWEdiTzY4X0NybjkwYXdXTnhyWG1kNEpHN3d3alZZaDRrUW9WV0ZqTTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJ3ZWJhcHBzIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6Im15c2VjcmV0bmFtZSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMDE5ZTBiYjMtMzI5NC00NDYyLTkyYzctOGM3YjBkYTkyZjBiIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OndlYmFwcHM6amVua2lucyJ9.anJ1zO_R6gpOSxcjFMHHQjX5JFLfabjuTmkcsRX4yeF6N5gNbQ-WhZVmhUJLzA7R274uef0JwJpzgRZX1GD1VlrHYSwS2Uppe5JmGLCmEInwpzogatXiiOneUJEneUoFgtCC1uEA9o6I9dttgbfEL_igx8J1Crs_wCDZbJ-xkDuwt1fKBjF92khLAowZVjU5lNmS5NfPqYC8qAt2j8l1i7rptFEf3eqLg_iu8n5Lwtf-ILCKaJMNRoesBVTkhH_HGfP56ERsmRaTAAm5sFjIU7vAU7hd-eYL-F53MMWwY44YiSjVX9mxwVuZRvv6gwUr1AtY3P7bynDaHuC52yCzVQ" // Replace this with the actual token
    }

    stages {
        stage('Setup Kubernetes Config') {
            steps {
                script {
                    // Create a kubeconfig file with the hard-coded token
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
