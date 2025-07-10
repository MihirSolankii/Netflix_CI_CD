pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images for frontend and backend...'
                sh 'docker compose build'
            }
        }

        stage('Start Services') {
            steps {
                echo 'Starting services with docker-compose...'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}
