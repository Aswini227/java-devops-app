pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'dockerhub-id'   // Your Jenkins Docker Hub credentials ID
        DOCKER_IMAGE = 'ashcreatingdocker/demo-app:latest'
    }

    stages {

        stage('Build with Maven (Docker)') {
            steps {
                // Use Maven Docker image to build your Java app
                sh """
                    docker run --rm \
                        -v \$(pwd):/app \
                        -v /var/jenkins_home/.m2:/root/.m2 \
                        -w /app \
                        maven:3.9.6-eclipse-temurin-17 \
                        mvn clean package
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build Docker image from current repo
                sh "docker build -t \$DOCKER_IMAGE ."
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push Docker image to Docker Hub using credentials
                withCredentials([usernamePassword(credentialsId: "\$DOCKER_HUB_CREDENTIALS",
                                                  usernameVariable: 'DOCKER_USER',
                                                  passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push \$DOCKER_IMAGE"
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}
