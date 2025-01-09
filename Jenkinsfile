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
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
    // Your AWS commands here, using AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY environment variables
    sh 'echo $AWS_ACCESS_KEY_ID' // Just for debugging, do not expose in production logs
    sh 'echo $AWS_SECRET_ACCESS_KEY' // Just for debugging, do not expose in production logs
}

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
