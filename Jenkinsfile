pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'oussama75/ci-cd-project'  // Docker Hub repo
        DOCKER_TAG = "${BUILD_NUMBER}"           // Auto-tag with Jenkins build number
    }

    stages {

        stage('Cloning Git') {
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

        stage('Test Image') {
            steps {
                echo "Testing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                
                // Run container in detached mode
                bat "docker run --rm -d --name test_container %DOCKER_IMAGE%:%DOCKER_TAG%"
                
                // Wait 5 seconds non-interactively
                bat "timeout /t 5 /nobreak"
                
                // Stop the container
                bat "docker stop test_container || exit 0"
            }
        }

        stage('Publish Image') {
            steps {
                echo "Pushing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} to Docker Hub..."
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-token', // Jenkins credential ID
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        // Login
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        
                        // Push the image
                        bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                        
                        // Also push "latest" tag
                        bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
                        bat "docker push %DOCKER_IMAGE%:latest"
                        
                        // Logout
                        bat 'docker logout'
                    }
                }
            }
        }

        stage('Deploy Image') {
            steps {
                echo "Deploying Docker image %DOCKER_IMAGE%:%DOCKER_TAG% locally..."
                // Stop any running container with the same name
                bat "docker stop deployed_container || exit 0"
                bat "docker rm deployed_container || exit 0"
                
                // Run the new container
                bat "docker run -d --name deployed_container -p 8080:80 %DOCKER_IMAGE%:%DOCKER_TAG%"
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
            // Cleanup and logout
            bat 'docker logout || exit 0'
        }
    }
}
