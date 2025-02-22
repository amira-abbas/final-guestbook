pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ahmed-ahmedd/guestbook.git'
            }
        }

        stage('Test Docker Environment') {
            steps {
                script {
                    sh 'docker --version'  // Check if Docker is installed
                    sh 'docker-compose --version'  // Check if Docker Compose is installed
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh 'docker-compose down || true'  // Stop old containers if running
                    sh 'docker-compose up -d'  // Start app in detached mode
                }
            }
        }
    }

    post {
        success {
            echo '🎉 Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed. Check logs!'
        }
    }
}
