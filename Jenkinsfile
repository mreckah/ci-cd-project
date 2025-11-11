pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'oussama75/ci-cd-project'
        DOCKER_TAG   = 'latest'
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
                echo "Pushing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',  // Use the Jenkins credentials ID you created
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Login to Docker Hub
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        
                        // Push the image
                        bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                        
                        // Logout to clean session
                        bat 'docker logout'
                    }
                }
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleaning up local Docker image...'
                bat "docker rmi %DOCKER_IMAGE%:%DOCKER_TAG% || exit 0"
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs!'
        }
        always {
            // Ensure logout even if failure happens
            bat 'docker logout || exit 0'
        }
    }
}
