pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
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
                    sh "docker build -t ${DOCKER_IMAGE}:latest . || echo 'Docker build failed!'"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_TOKEN')]) {
                        sh "docker login -u your-docker-username -p $DOCKER_TOKEN"
                        sh "docker tag ${DOCKER_IMAGE}:latest your-docker-username/${DOCKER_IMAGE}:latest"
                        sh "docker push your-docker-username/${DOCKER_IMAGE}:latest"
                    }
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

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    if [ -f test.sh ]; then
                        chmod +x test.sh
                        ./test.sh || echo "Tests failed!"
                    else
                        echo "⚠️ No test script found!"
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
