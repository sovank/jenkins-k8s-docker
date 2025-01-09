pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
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
                    withCredentials([string(credentialsId: 'aws_credentials', variable: 'AWS_ACCESS_KEY_ID'), 
                                     string(credentialsId: 'aws_credentials', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        dir('kubernetes') {
                            sh "aws eks update-kubeconfig --name Jenkins-k8s"
                            sh "kubectl apply -f deployment.yaml"
                            sh "kubectl apply -f service.yaml"
                        }
                    }
                }
            }
        }
    }
}
