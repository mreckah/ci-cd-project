pipeline {
    agent any

    environment {
        registry = "oussama75/tp4"         // Ton compte DockerHub
        registryCredential = 'dockerhub'    // ID des credentials Jenkins pour DockerHub
        dockerImage = ''
    }

    stages {
        stage('Cloning Git') {
            steps {
                echo "Cloning repository..."
                git url: 'https://github.com/mreckah/ci-cd-project', branch: 'main'
            }
        }

        stage('Building image') {
            steps {
                script {
                    echo "Building Docker image..."
                    dockerImage = docker.build("${registry}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Test image') {
            steps {
                script {
                    echo "Running tests on Docker image..."
                    // Exemple basique : tu peux remplacer par de vrais tests
                    dockerImage.inside {
                        sh 'echo "Tests passed inside container"'
                    }
                }
            }
        }

        stage('Publish Image') {
            steps {
                script {
                    echo "Publishing Docker image to DockerHub..."
                    docker.withRegistry('https://index.docker.io/v1/', registryCredential) {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest") // Pousse aussi le tag latest
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
