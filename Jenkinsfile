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
        
        stage('Verify Docker') {
            steps {
                echo 'Verifying Docker installation...'
                sh '''
                    docker --version
                    docker compose version
                    docker info
                '''
            }
        }
        
        stage('Stop Existing Services') {
            steps {
                echo 'Stopping any existing services...'
                sh '''
                    docker compose down || true
                    docker ps -a --format "table {{.Names}}" | grep -E "(netflix|frontend|backend)" | xargs -r docker rm -f || true
                '''
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh '''
                    if [ ! -f docker-compose.yml ]; then
                        echo "Error: docker-compose.yml not found!"
                        ls -la
                        exit 1
                    fi
                    
                    echo "Building images..."
                    docker compose build --no-cache
                '''
            }
        }
        
        stage('Start Services') {
            steps {
                echo 'Starting services...'
                sh '''
                    docker compose up -d
                    echo "Waiting for services to start..."
                    sleep 30
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                echo 'Performing health check...'
                sh '''
                    docker compose ps
                    docker ps
                '''
            }
        }
    }
    stage('Start Netdata') {
    steps {
        echo 'Starting Netdata monitoring...'
        sh '''
            docker compose up -d netdata
            echo "Waiting for Netdata to initialize..."
            sleep 10
        '''
    }
}

    
    post {
        always {
            echo 'Pipeline execution completed'
            sh '''
                echo "=== Final Container Status ==="
                docker ps || true
            '''
        }
        success {
            echo 'Pipeline completed successfully! 🎉'
            sh '''
                echo "=== Application is running ==="
                docker compose ps || true
            '''
        }
        failure {
            echo 'Pipeline failed. Checking logs...'
            sh '''
                echo "=== Container Logs ==="
                docker compose logs || true
                echo "=== Cleaning up ==="
                docker compose down || true
            '''
        }
    }
}

