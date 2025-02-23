pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"  // Docker image name
        SONARQUBE_SERVER = "SonarQube"   // SonarQube server name from Jenkins settings
        SONARQUBE_PROJECT_KEY = "final-guestbook"  // Unique SonarQube project key
        SONAR_HOST_URL = "http://16.170.182.27:9000"  // SonarQube URL
        SONAR_TOKEN = credentials('sonarqube-token')  // SonarQube authentication token
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

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=${SONAR_TOKEN}
                        '''
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 2, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: true
                    }
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
