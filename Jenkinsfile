pipeline {
    agent any
    
    environment {
        WEB_PORT = "18085"
        DB_PORT = "13308"
        PHPMYADMIN_PORT = "18086"
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout stage running'
                git branch: 'main', url: 'https://github.com/JangoBernardofficial/bernard.git'
            }
        }
        
        stage('Clean Workspace') {
            steps {
                echo 'Cleanup workspace stage running'
                sh '''
                    echo "🧹 Cleaning up previous runs..."
                    docker-compose down -v --remove-orphans 2>/dev/null || true
                    docker system prune -f 2>/dev/null || true
                    
                    # Kill any processes on our ports
                    sudo fuser -k ${WEB_PORT}/tcp 2>/dev/null || true
                    sudo fuser -k ${DB_PORT}/tcp 2>/dev/null || true
                    sudo fuser -k ${PHPMYADMIN_PORT}/tcp 2>/dev/null || true
                '''
            }
        }
        
        stage('Build') {
            steps {
                echo 'Build stage running'
                sh 'docker-compose build --no-cache'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Test stage running'
                script {
                    // Verify the correct docker-compose.yml is being used
                    sh '''
                        echo "=== VERIFYING DOCKER-COMPOSE.YML ==="
                        pwd
                        ls -la docker-compose.yml
                        echo "=== CONTENT ==="
                        cat docker-compose.yml
                        echo "=== PORTS CONFIGURATION ==="
                        grep -A 5 "ports:" docker-compose.yml
                    '''
                    
                    // Start services
                    sh 'docker-compose up -d'
                    
                    // Wait for services
                    sleep time: 45, unit: 'SECONDS'
                    
                    // Check container status
                    sh '''
                        echo "=== CONTAINER STATUS ==="
                        docker-compose ps
                        echo "=== WEB CONTAINER LOGS ==="
                        docker-compose logs web
                    '''
                    
                    // Test web server on the correct port
                    sh """
                        echo "Testing web server on port ${WEB_PORT}..."
                        curl -v --retry 3 --retry-delay 5 http://localhost:${WEB_PORT}
                    """
                    
                    // Test database
                    sh '''
                        echo "Testing database..."
                        sleep 10
                        docker-compose exec -T db mysql -u root -ppassword -e "USE 221010123_shareride_db; SHOW TABLES;"
                    '''
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploy stage running'
                sh """
                    echo "✅ DEPLOYMENT SUCCESSFUL"
                    echo "🌐 Application: http://localhost:${WEB_PORT}"
                    echo "🗄️ Database Admin: http://localhost:${PHPMYADMIN_PORT}"
                """
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleanup stage running'
                sh 'docker-compose down -v'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
            sh 'docker-compose down -v 2>/dev/null || true'
        }
        success {
            echo '🎉 Pipeline succeeded!'
        }
        failure {
            echo '❌ Pipeline failed!'
            sh '''
                echo "=== DEBUG INFO ==="
                docker ps -a
                docker network ls
                echo "=== PORT USAGE ==="
                netstat -tuln | grep -E ":(8080|18085|13308|18086)" || true
            '''
        }
    }
}
