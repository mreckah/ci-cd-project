pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'oussama75/ci-cd-project'  // Docker Hub repo
        DOCKER_TAG = "${BUILD_NUMBER}"           // Auto-tag with Jenkins build number
    }

    stages {
        // 1️⃣ Cloning Git
        stage('Cloning Git') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        // 2️⃣ Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        // 3️⃣ Test Image
        stage('Test Image') {
            steps {
                echo "Testing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."

                // Remove any previous container with the same name
                bat "docker stop test_container || exit 0"
                bat "docker rm test_container || exit 0"

                // Run container
                bat "docker run --rm -d --name test_container %DOCKER_IMAGE%:%DOCKER_TAG%"

                // Wait 5 seconds for container to start
                bat "timeout /t 5 /nobreak"

                // Stop test container
                bat "docker stop test_container || exit 0"
            }
        }

        // 4️⃣ Publish Image to Docker Hub
        stage('Publish Image') {
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

                        // Push build number tag
                        bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"

                        // Tag and push latest
                        bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
                        bat "docker push %DOCKER_IMAGE%:latest"

                        // Logout
                        bat 'docker logout'
                    }
                }
            }
        }

        // 5️⃣ Deploy Image (to Docker Engine)
        stage('Deploy Image') {
            steps {
                echo "Deploying Docker image %DOCKER_IMAGE%:%DOCKER_TAG%..."

                // Remove any previous container deployed
                bat "docker stop deployed_container || exit 0"
                bat "docker rm deployed_container || exit 0"

                // Run container
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
            // Cleanup: remove any leftover test container
            bat "docker stop test_container || exit 0"
            bat "docker rm test_container || exit 0"
            bat 'docker logout || exit 0'
        }
    }
}
