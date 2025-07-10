pipeline {
    agent any
    
    environment {
        DOCKER_HOST = 'unix:///var/run/docker.sock'
    }
    
    stages {
        stage('Install Docker CLI') {
            steps {
                script {
                    // Check if Docker CLI is already installed
                    def dockerInstalled = sh(script: 'which docker', returnStatus: true) == 0
                    if (!dockerInstalled) {
                        echo 'Installing Docker CLI...'
                        sh '''
                            # Update package manager
                            apt-get update
                            
                            # Install required packages
                            apt-get install -y \
                                apt-transport-https \
                                ca-certificates \
                                curl \
                                gnupg \
                                lsb-release
                            
                            # Add Docker's official GPG key
                            curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
                            
                            # Add Docker repository
                            echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
                            
                            # Update package database
                            apt-get update
                            
                            # Install Docker CLI only (not daemon since we're using host's daemon)
                            apt-get install -y docker-ce-cli docker-compose-plugin
                            
                            # Verify installation
                            docker --version
                            docker compose version
                        '''
                    } else {
                        echo 'Docker CLI already installed'
                        sh '''
                            docker --version
                            docker compose version || echo "Docker Compose plugin not found"
                        '''
                    }
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                echo 'Cloning repository from main branch...'
                git branch: 'main', 
                    url: 'https://github.com/MihirSolankii/Netflix_CI_CD'
            }
        }
        
        stage('Check Docker Connection') {
            steps {
                echo 'Testing Docker connection...'
                sh '''
                    docker info
                    echo "Docker is working correctly!"
                '''
            }
        }
        
        stage('Stop Existing Services') {
            steps {
                echo 'Stopping any existing services...'
                sh '''
                    # Stop existing containers if they exist
                    docker compose down || true
                    
                    # Remove any existing containers with the same name
                    docker ps -a --format "table {{.Names}}" | grep -E "(netflix|frontend|backend)" | xargs -r docker rm -f || true
                '''
            }
        }
        
        stage('Build Docker Images') {
            steps {
                echo 'Building Docker images...'
                sh '''
                    # Check if docker-compose.yml exists
                    if [ ! -f docker-compose.yml ]; then
                        echo "Error: docker-compose.yml not found!"
                        exit 1
                    fi
                    
                    # Build images
                    docker compose build --no-cache
                '''
            }
        }
        
        stage('Start Services') {
            steps {
                echo 'Starting services...'
                sh '''
                    # Start services in detached mode
                    docker compose up -d
                    
                    # Wait for services to start
                    echo "Waiting for services to start..."
                    sleep 30
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                echo 'Performing health check...'
                sh '''
                    # Check running containers
                    docker compose ps
                    
                    # Show container logs (last 20 lines)
                    docker compose logs --tail=20
                    
                    # List all running containers
                    docker ps
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh '''
                echo "=== Container Status ==="
                docker ps -a
                echo "=== Docker Compose Status ==="
                docker compose ps || true
            '''
        }
        success {
            echo 'Pipeline completed successfully! 🎉'
            sh '''
                echo "=== Application is running ==="
                docker compose ps
            '''
        }
        failure {
            echo 'Pipeline failed. Checking logs...'
            sh '''
                echo "=== Container Logs ==="
                docker compose logs || true
                echo "=== Cleaning up failed containers ==="
                docker compose down || true
            '''
        }
        cleanup {
            echo 'Performing cleanup...'
            sh '''
                # Clean up unused images and containers
                docker system prune -f || true
            '''
        }
    }
}
