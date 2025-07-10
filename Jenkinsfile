// Option 1: Robust Pipeline with Error Handling
pipeline {
    agent any
    
    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        PROJECT_NAME = 'netflix-cicd'
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                cleanWs()
            }
        }
        
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository from main branch...'
                git branch: 'main', 
                    url: 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }
        
        stage('Check Prerequisites') {
            steps {
                echo 'Checking if Docker and Docker Compose are available...'
                sh '''
                    docker --version
                    docker-compose --version || docker compose version
                '''
            }
        }
        
        stage('Stop Existing Services') {
            steps {
                echo 'Stopping any existing services...'
                sh '''
                    docker-compose -f ${DOCKER_COMPOSE_FILE} down || true
                    docker compose -f ${DOCKER_COMPOSE_FILE} down || true
                '''
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh '''
                    if [ -f docker-compose.yml ]; then
                        docker compose build --no-cache
                    else
                        echo "docker-compose.yml not found!"
                        exit 1
                    fi
                '''
            }
        }
        
        stage('Start Services') {
            steps {
                echo 'Starting services with docker-compose...'
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
                    echo "Services status checked"
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh '''
                echo "Listing running containers:"
                docker ps
            '''
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Cleaning up...'
            sh '''
                docker compose logs || true
                docker compose down || true
            '''
        }
        cleanup {
            echo 'Performing cleanup...'
            sh '''
                docker system prune -f || true
            '''
        }
    }
}

// Option 2: Simple Pipeline (if you prefer minimal approach)
pipeline {
    agent any
    
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }
        
        stage('Build and Deploy') {
            steps {
                sh '''
                    echo "Building Docker images..."
                    docker compose build
                    
                    echo "Stopping existing containers..."
                    docker compose down || true
                    
                    echo "Starting new containers..."
                    docker compose up -d
                    
                    echo "Checking container status..."
                    docker compose ps
                '''
            }
        }
    }
    
    post {
        failure {
            sh 'docker compose logs'
        }
    }
}

// Option 3: Pipeline with Credentials (if you need authentication)
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = credentials('docker-registry-credentials')
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'main', 
                    credentialsId: 'github-credentials',
                    url: 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }
        
        stage('Build Images') {
            steps {
                sh '''
                    echo "Building images..."
                    docker compose build
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                sh '''
                    echo "Deploying application..."
                    docker compose down || true
                    docker compose up -d
                '''
            }
        }
    }
}
