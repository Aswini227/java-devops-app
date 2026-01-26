pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ashcreatingdocker/demo-app:latest'
        KUBECONFIG = '/home/ubuntu/.kube/config'  // path to kubeconfig for EKS access
    }

    stages {
        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Aswini227/java-devops-app.git'
            }
        }

        stage('Build with Maven (Docker)') {
            steps {
                sh """
                docker run --rm \
                  -v \$WORKSPACE:/app \
                  -v /root/.m2:/root/.m2 \
                  -w /app maven:3.9.6-eclipse-temurin-17 \
                  mvn clean package
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t \$DOCKER_IMAGE ."
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-id',  // <-- your Docker Hub credentials ID
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                      echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                      docker push \$DOCKER_IMAGE
                    """
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                // Apply all YAMLs inside your k8s folder
                sh 'kubectl apply -f k8s/'

                // Wait until deployment is rolled out
                sh 'kubectl rollout status deployment/demo-app'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully! Your app is now live on EKS.'
        }
        failure {
            echo '❌ Pipeline failed! Check the logs above.'
        }
    }
}
