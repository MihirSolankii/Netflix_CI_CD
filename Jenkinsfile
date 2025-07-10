pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository from main branch...'
                git branch: 'main', 
                    url: 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }
        
        stage('Build and Deploy') {
            steps {
                echo 'Building and deploying Netflix application...'
                sh '''
                    # Check Docker
                    docker --version
                    docker compose version
                    
                    # Stop existing services
                    docker compose down || true
                    
                    # Build images
                    docker compose build --no-cache
                    
                    # Start services
                    docker compose up -d
                    
                    # Wait and check status
                    sleep 30
                    docker compose ps
                '''
            }
        }
    }
    
    post {
        always {
            sh 'docker ps'
        }
        failure {
            sh 'docker compose logs'
        }
    }
}
