pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'oussama75/ci-cd-project'  // Docker Hub repo
        DOCKER_TAG = "${BUILD_NUMBER}"           // Auto-tag with Jenkins build number
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo "Pushing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} to Docker Hub..."
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-token', // Your Jenkins credential ID
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Login
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        
                        // Push image
                        bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                        
                        // Optionally push "latest" tag
                        bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
                        bat "docker push %DOCKER_IMAGE%:latest"
                        
                        // Logout
                        bat 'docker logout'
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo 'Cleaning up local Docker images...'
                bat "docker rmi %DOCKER_IMAGE%:%DOCKER_TAG% || exit 0"
                bat "docker rmi %DOCKER_IMAGE%:latest || exit 0"
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            // Ensure Docker logout even if failure occurs
            bat 'docker logout || exit 0'
        }
    }
}
