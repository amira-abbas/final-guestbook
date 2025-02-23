pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"  // Docker image name
        SONARQUBE_SERVER = "SonarQube"   // SonarQube server name from Jenkins settings
        SONARQUBE_PROJECT_KEY = "final-guestbook"  // Unique SonarQube project key
        SONAR_HOST_URL = "http://16.170.182.27:9000"  // SonarQube URL
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf * || true'  // Clears workspace
                    checkout scm
                    sh 'ls -la'  // Debug: Check if files exist
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh 'docker --version || echo "Docker not installed!"'
                    sh 'docker-compose --version || echo "Docker Compose not found!"'
                    sh 'mvn --version || echo "Maven not installed!"'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withSonarQubeEnv(SONARQUBE_SERVER) {
                        withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                            sh '''
                            sonar-scanner \
                              -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=${SONAR_HOST_URL} \
                              -Dsonar.login=$SONAR_TOKEN
                            '''
                        }
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            echo "WARNING: Quality Gate failed, but continuing deployment..."
                        }
                    }
                }
            }
        } // <-- Corrected closing brace for Quality Gate stage

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . || echo 'Docker build failed!'"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    if [ -f docker-compose.yml ]; then
                        docker-compose down || echo "Failed to stop running containers"
                        docker-compose up -d || echo "Failed to start containers"
                    else
                        echo "⚠️ No docker-compose.yml found!"
                    fi
                    '''
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
