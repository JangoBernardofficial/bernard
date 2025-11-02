pipeline {
    agent any
    
    environment {
        // Use environment variables for ports to avoid conflicts
        WEB_PORT = "8085"
        DB_PORT = "3308"
        PHPMYADMIN_PORT = "8086"
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
                    # Clean up any existing containers
                    docker-compose down -v 2>/dev/null || true
                    docker system prune -f 2>/dev/null || true
                    
                    # Kill any processes using our ports
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
                    // Start services
                    sh 'docker-compose up -d'
                    
                    // Wait for services to be ready
                    sleep time: 45, unit: 'SECONDS'
                    
                    // Test web server
                    sh "curl -f http://localhost:${WEB_PORT} || exit 1"
                    
                    // Test database connectivity
                    sh 'docker-compose exec -T db mysql -u root -ppassword -e "USE 221010123_shareride_db; SHOW TABLES;"'
                    
                    // Test health endpoint
                    sh "curl -f http://localhost:${WEB_PORT}/health.php || exit 1"
                }
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploy stage running'
                sh '''
                    echo "Application deployed successfully!"
                    echo "Web Application: http://localhost:${WEB_PORT}"
                    echo "phpMyAdmin: http://localhost:${PHPMYADMIN_PORT}"
                '''
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
            sh '''
                # Always cleanup
                docker-compose down -v 2>/dev/null || true
                docker system prune -f 2>/dev/null || true
            '''
            script {
                // Archive test results if any
                junit '**/test-results/*.xml' 
            }
        }
        success {
            echo 'Pipeline succeeded! 🎉'
            sh '''
                echo "✅ All tests passed!"
                echo "🌐 Application URL: http://localhost:${WEB_PORT}"
                echo "🗄️ phpMyAdmin URL: http://localhost:${PHPMYADMIN_PORT}"
            '''
        }
        failure {
            echo 'Pipeline failed! ❌'
            sh '''
                echo "Debug information:"
                docker-compose ps 2>/dev/null || true
                docker-compose logs web 2>/dev/null || true
            '''
        }
    }
}
