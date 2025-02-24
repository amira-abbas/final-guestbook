pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        BUILD_LOG_FILE = "build_output.log"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf * || true'  
                    checkout scm
                    sh 'ls -la'
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh 'docker --version || echo "Docker not installed!"'
                    sh 'docker-compose --version || echo "Docker Compose not found!"'
                    sh 'sonar-scanner --version || echo "SonarScanner not installed!"'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=$SONAR_TOKEN \
                          -Dsonar.qualitygate.wait=true \
                          -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**"
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . > ${BUILD_LOG_FILE} 2>&1 || echo 'Docker build failed!'"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    if [ -f docker-compose.yml ]; then
                        docker-compose down || echo "Failed to stop running containers" >> ${BUILD_LOG_FILE}
                        docker-compose up -d || echo "Failed to start containers" >> ${BUILD_LOG_FILE}
                    else
                        echo "⚠️ No docker-compose.yml found!" >> ${BUILD_LOG_FILE}
                    fi
                    '''
                }
            }
        }

        stage('Ensure Jenkins and Project Containers Auto-Start') {
            steps {
                script {
                    sh '''
                    echo "Ensuring Jenkins container restarts on boot..."
                    docker update --restart always jenkins || echo "Failed to update Jenkins container"
                    echo "Ensuring project containers restart on boot..."
                    docker update --restart always $(docker ps -q) || echo "Failed to update project containers"
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
