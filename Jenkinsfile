pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checkout stage running'
                git branch: 'main', url: 'https://github.com/JangoBernardofficial/bernard.git'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Build stage running'
                sh 'docker-compose build'
            }
        }
        
        stage('Test') {
            steps {
                echo 'Test stage running'
                sh 'docker-compose up -d'
                sleep time: 30, unit: 'SECONDS'
                sh 'curl -f http://localhost:8080 || exit 1'
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploy stage running'
                // Additional deployment steps can be added here
            }
        }
        
        stage('Cleanup') {
            steps {
                echo 'Cleanup stage running'
                sh 'docker-compose down'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline completed'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
