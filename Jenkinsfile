pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'oussama75/ci-cd-project'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Cloning Git') {
            steps {
                echo 'Cloning repository...'
                checkout scm
            }
        }

        stage('Building Docker Image') {
            steps {
                echo "Building Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
            }
        }

        stage('Test Image') {
            steps {
                echo "Testing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG}..."
                // Lancer un container en arrière-plan pour test puis supprimer
                bat "docker run --rm -d --name test_container %DOCKER_IMAGE%:%DOCKER_TAG%"
                // Petite pause pour vérifier si container démarre
                bat "timeout /t 5"
                // Vérifier si le container est en cours d'exécution
                bat "docker ps -a"
                // Supprimer le container de test
                bat "docker stop test_container || exit 0"
            }
        }

        stage('Publish Image') {
            steps {
                echo "Pushing Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} to Docker Hub..."
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-hub-token', 
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        bat 'echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin'
                        bat "docker push %DOCKER_IMAGE%:%DOCKER_TAG%"
                        bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% %DOCKER_IMAGE%:latest"
                        bat "docker push %DOCKER_IMAGE%:latest"
                        bat 'docker logout'
                    }
                }
            }
        }

        stage('Deploy Image') {
            steps {
                echo "Deploying Docker image ${DOCKER_IMAGE}:${DOCKER_TAG} on local Docker Engine..."
                // Supprimer container existant si présent
                bat "docker rm -f deployed_container || exit 0"
                // Lancer le container avec port mapping
                bat "docker run -d --name deployed_container -p 8080:80 %DOCKER_IMAGE%:%DOCKER_TAG%"
                bat "docker ps -a"
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
            bat 'docker logout || exit 0'
        }
    }
}
