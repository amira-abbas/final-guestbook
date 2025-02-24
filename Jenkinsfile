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
                    sh '''
                    echo "Starting Cleanup..." | tee $OUTPUT_LOG
                    rm -rf * || true
                    rm -f $OUTPUT_LOG || true
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                    sh '''
                    echo "Listing checked-out files..." | tee -a $OUTPUT_LOG
                    ls -la | tee -a $OUTPUT_LOG
                    '''
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                    echo "Checking environment..." | tee -a $OUTPUT_LOG
                    docker --version | tee -a $OUTPUT_LOG || echo "Docker not installed!" | tee -a $OUTPUT_LOG
                    docker-compose --version | tee -a $OUTPUT_LOG || echo "Docker Compose not found!" | tee -a $OUTPUT_LOG
                    sonar-scanner --version | tee -a $OUTPUT_LOG || echo "SonarScanner not installed!" | tee -a $OUTPUT_LOG
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        echo "Starting SonarQube Analysis..." | tee -a $OUTPUT_LOG
                        sonar-scanner \
                          -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=$SONAR_TOKEN \
                          -Dsonar.qualitygate.wait=true \
                          -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**" | tee -a $OUTPUT_LOG
                        '''
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                    echo "Building Docker image..." | tee -a $OUTPUT_LOG
                    docker build -t ${DOCKER_IMAGE}:latest . | tee -a $OUTPUT_LOG || echo 'Docker build failed!' | tee -a $OUTPUT_LOG
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]) {
                        sh '''
                        echo "Logging into Docker Hub..." | tee -a $OUTPUT_LOG
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin
                        docker tag ${DOCKER_IMAGE}:latest $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest
                        docker push $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest | tee -a $OUTPUT_LOG
                        '''
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    echo "Deploying Application..." | tee -a $OUTPUT_LOG
                    if [ -f docker-compose.yml ]; then
                        docker-compose down | tee -a $OUTPUT_LOG || echo "Failed to stop running containers" | tee -a $OUTPUT_LOG
                        docker-compose up -d | tee -a $OUTPUT_LOG || echo "Failed to start containers" | tee -a $OUTPUT_LOG
                    else
                        echo "⚠️ No docker-compose.yml found!" | tee -a $OUTPUT_LOG
                    fi
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    echo "Running application tests..." | tee -a $OUTPUT_LOG
                    if [ -f tests/run-tests.sh ]; then
                        chmod +x tests/run-tests.sh
                        ./tests/run-tests.sh | tee -a $OUTPUT_LOG
                    else
                        echo "⚠️ No test script found!" | tee -a $OUTPUT_LOG
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
