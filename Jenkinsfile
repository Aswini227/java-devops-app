pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ashcreatingdocker/demo-app:latest'
        KUBECONFIG   = '/var/lib/jenkins/.kube/config'
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Aswini227/java-devops-app.git'
            }
        }

        stage('Build with Maven (Docker)') {
            steps {
                sh '''
                docker run --rm \
                  -v $WORKSPACE:/app \
                  -v /var/lib/jenkins/.m2:/root/.m2 \
                  -w /app maven:3.9.6-eclipse-temurin-17 \
                  mvn clean package
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-id',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                      echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                      docker push $DOCKER_IMAGE
                    '''
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                echo "Using kubeconfig at: $KUBECONFIG"
                kubectl get nodes
                kubectl apply -f k8s/
                kubectl rollout status deployment/demo-app
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully! App is live on EKS 🚀'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs.'
        }
    }
}
