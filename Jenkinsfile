pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        AWS_ACCESS_KEY_ID = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
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
                    // Retrieve Docker Hub credentials
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
                    dir('kubernetes') {
                        // Update packages inside the cluster
                        sh "aws eks update-kubeconfig --name Jenkins-k8s"
                        // Deploy an application
                        sh "kubectl apply -f deployment.yaml"
                        // Deploy a service
                        sh "kubectl apply -f service.yaml"
                    }
                }
            }
        }
    }
}
