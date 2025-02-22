pipeline {
    agent any

    environment {
        SONARQUBE_PROJECT_KEY = 'guestbook'  // Set SonarQube project key
        SONARQUBE_URL = 'http://localhost:9000'  // SonarQube server URL
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/ahmed-ahmedd/guestbook.git'
            }
        }

        stage('Static Code Analysis (SAST)') {
            steps {
                withSonarQubeEnv('SonarQube') { // Use configured SonarQube in Jenkins
                    sh """
                        sonar-scanner \
                        -Dsonar.projectKey=$SONARQUBE_PROJECT_KEY \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONARQUBE_URL
                    """
                }
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

        stage('Run Ansible Tests') {
            steps {
                script {
                    sh '''
                    ansible --version
                    ansible-playbook -i inventory test-playbook.yml
                    '''
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh 'docker-compose down'  // Stop old containers
                    sh 'docker-compose up -d' // Start app in detached mode
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
