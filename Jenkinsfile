pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"  // Set Docker image name
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf *'  // ⚠️ Clears workspace to avoid conflicts
                    checkout([$class: 'GitSCM',
                        branches: [[name: '*/main']], // Ensure correct branch
                        userRemoteConfigs: [[
                            url: 'https://github.com/ahmed-ahmedd/final-guestbook.git',
                            credentialsId: 'github-ssh-key'
                        ]]
                    ])
                }
                sh 'ls -la'  // Debug: Check if all files exist
            }
        }

        stage('Test Docker Environment') {
            steps {
                script {
                    sh 'docker --version'
                    sh 'docker-compose --version'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest ."
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh 'docker-compose down || true'  // Stop running containers (ignore errors)
                    sh 'docker-compose up -d'  // Start application
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed. Check logs!"
        }
    }
}
