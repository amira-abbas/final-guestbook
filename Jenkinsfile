pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        LOG_FILE = "build_output.log"  // Define log file name
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf * || true'
                    checkout scm
                    sh 'ls -la | tee -a ${LOG_FILE}'
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                    docker --version | tee -a ${LOG_FILE} || echo "Docker not installed!" | tee -a ${LOG_FILE}
                    docker-compose --version | tee -a ${LOG_FILE} || echo "Docker Compose not found!" | tee -a ${LOG_FILE}
                    sonar-scanner --version | tee -a ${LOG_FILE} || echo "SonarScanner not installed!" | tee -a ${LOG_FILE}
                    '''
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
                          -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**" \
                        | tee -a ${LOG_FILE}
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . | tee -a ${LOG_FILE}"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    if [ -f docker-compose.yml ]; then
                        docker-compose down | tee -a ${LOG_FILE} || echo "Failed to stop running containers" | tee -a ${LOG_FILE}
                        docker-compose up -d | tee -a ${LOG_FILE} || echo "Failed to start containers" | tee -a ${LOG_FILE}
                    else
                        echo "⚠️ No docker-compose.yml found!" | tee -a ${LOG_FILE}
                    fi
                    '''
                }
            }
        }
    }

    post {
        always {
            script {
                echo "📄 Build output log:"
                sh "cat ${LOG_FILE}"
            }
            archiveArtifacts artifacts: 'build_output.log', fingerprint: true
        }
        success {
            echo "✅ Deployment Successful!"
        }
        failure {
            echo "❌ Deployment Failed. Check logs!"
        }
    }
}
