pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ashcreatingdocker/demo-app:latest'
        K8S_FOLDER = 'k8s'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Aswini227/java-devops-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f $K8S_FOLDER/'
            }
        }
    }

    post {
        success {
            echo "Deployment to EKS Successful! 🚀"
        }
        failure {
            echo "Something went wrong! ❌"
        }
    }
}
