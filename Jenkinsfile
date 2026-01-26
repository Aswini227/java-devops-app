pipeline {
    agent any

    environment {  
        DOCKER_IMAGE = 'ashcreatingdocker/demo-app:latest'
        KUBECONFIG = '/home/ubuntu/.kube/config'  // Jenkins will use this to access EKS
    }

    stages {  
        stage('Clone Code') {  
            steps {  
                git branch: 'main', url: 'https://github.com/Aswini227/java-devops-app.git'  
            }  
        }  

        stage('Build with Maven') {  
            steps {  
                // Build Maven directly on EC2 workspace so target/demo-0.0.1-SNAPSHOT.jar exists
                sh 'mvn clean package -DskipTests'  
            }  
        }  

        stage('Build Docker Image') {  
            steps {  
                sh "docker build -t $DOCKER_IMAGE ."  
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
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE
                    """  
                }  
            }  
        }  

        stage('Deploy to EKS') {  
            steps {  
                // Make sure Jenkins has access to kubeconfig on EC2
                sh 'kubectl apply -f k8s/'  
                sh 'kubectl rollout status deployment/demo-app'  // wait for pod to be ready
            }  
        }  
    }  

    post {  
        success {  
            echo '✅ Pipeline completed successfully! Your app is live on EKS!'  
        }  
        failure {  
            echo '❌ Pipeline failed! Check logs for details.'  
        }  
    }  
}
