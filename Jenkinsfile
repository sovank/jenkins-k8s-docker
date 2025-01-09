pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        // Specify where kubeconfig should be stored if needed, especially in multi-user environments or where default location permissions might be restricted
        KUBECONFIG = "/home/ubuntu/.kube/config"
    }

    stages {
        stage('Checkout Source') {
            steps {
                git branch: 'main', url: 'https://github.com/scaler-bhavya/jenkins-docker.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    sh 'docker build -t bhavyascaler/react-app:latest .'
                }
            }
        }

        stage('Login') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'DOCKERHUB_PSW', usernameVariable: 'DOCKERHUB_USR')]) {
                        sh 'echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin'
                    }
                }
            }
        }

        stage('Push') {
            steps {
                sh 'docker push bhavyascaler/react-app:latest'
            }
        }

        stage("Deploy to EKS") {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials', accessKeyVariable: 'AWS_ACCESS_KEY_ID', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                        // Explicitly setting the region and specifying the kubeconfig path ensures that the AWS CLI and kubectl use correct configurations
                        sh "aws eks update-kubeconfig --name Jenkins-k8s --region ${AWS_DEFAULT_REGION} --kubeconfig ${KUBECONFIG}"
                        dir('kubernetes') {
                            sh "kubectl apply -f deployment.yaml --kubeconfig ${KUBECONFIG}"
                            sh "kubectl apply -f service.yaml --kubeconfig ${KUBECONFIG}"
                        }
                    }
                }
            }
        }
    }
}
