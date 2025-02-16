pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/sovank/jenkins-k8s-docker'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    sh 'docker build -t sovankeshri/react-app:latest .' // Or your preferred tag
                }
            }
        }

        stage('Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'sovan-cred', passwordVariable: 'DOCKERHUB_PSW', usernameVariable: 'DOCKERHUB_USR')]) {
                        sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                    }
                }
            }
        }

        stage('Push') {
            steps {
                sh 'docker push sovankeshri/react-app:latest' // Make sure this matches your build tag
            }
        }

        stage("Deploy to Kind") {
            steps {
                withCredentials([file(credentialsId: 'kube-config', variable: 'KUBECONFIG_FILE')]) {
                    script {
                        // 1. Set Kind Context (if needed) - Important!
                        sh 'kubectl config use-context kind-my-project-cluster --kubeconfig "$KUBECONFIG_FILE"'
    
                        // 2. Load Image into Kind (if needed) -  Crucial for local Kind
                        sh 'kind load docker-image sovankeshri/react-app:latest --name my-project-cluster'
    
                        // 3. Apply Kubernetes Manifests
                        sh 'kubectl apply -f deployment.yaml --kubeconfig "$KUBECONFIG_FILE"'
                        sh 'kubectl apply -f service.yaml --kubeconfig "$KUBECONFIG_FILE"'
    
                        // Optional: Verify Deployment
                        sh 'kubectl get pods --kubeconfig "$KUBECONFIG_FILE"'
                        sh 'kubectl get service --kubeconfig "$KUBECONFIG_FILE"'
                    }
                }
            }
        }
    }
}
