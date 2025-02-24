pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        DOCKER_HUB_USERNAME = "ahmedelshandidy"
        OUTPUT_LOG = "pipeline_output.log"
    }

    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh 'echo "Starting Cleanup..." > $OUTPUT_LOG'
                    sh 'rm -rf * || true'  
                    sh 'rm -f $OUTPUT_LOG || true'  // Clears the output log file if it exists
                }
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                    sh 'ls -la >> $OUTPUT_LOG'  // Logs output to file
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh 'docker --version || echo "Docker not installed!" >> $OUTPUT_LOG'
                    sh 'docker-compose --version || echo "Docker Compose not found!" >> $OUTPUT_LOG'
                    sh 'sonar-scanner --version || echo "SonarScanner not installed!" >> $OUTPUT_LOG'
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
                          -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**" >> $OUTPUT_LOG
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . >> $OUTPUT_LOG || echo 'Docker build failed!' >> $OUTPUT_LOG"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]) {
                        sh '''
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin
                        docker tag ${DOCKER_IMAGE}:latest $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest
                        docker push $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest >> $OUTPUT_LOG
                        '''
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    if [ -f docker-compose.yml ]; then
                        docker-compose down || echo "Failed to stop running containers" >> $OUTPUT_LOG
                        docker-compose up -d || echo "Failed to start containers" >> $OUTPUT_LOG
                    else
                        echo "⚠️ No docker-compose.yml found!" >> $OUTPUT_LOG
                    fi
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    echo "Running application tests..." >> $OUTPUT_LOG
                    if [ -f tests/run-tests.sh ]; then
                        chmod +x tests/run-tests.sh
                        ./tests/run-tests.sh >> $OUTPUT_LOG
                    else
                        echo "⚠️ No test script found!" >> $OUTPUT_LOG
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
