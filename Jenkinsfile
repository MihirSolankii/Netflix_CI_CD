pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yaml'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/MihirSolankii/Netflix_CI_CD', branch: 'main'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images for frontend and backend...'
                sh 'docker-compose build'
            }
        }

        stage('Start Services') {
            steps {
                echo 'Starting Docker containers...'
                sh 'docker-compose up -d'
            }
        }
    }

    post {
        success {
            echo 'Application deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
