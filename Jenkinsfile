pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        OUTPUT_FILE = "pipeline.log"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    sh 'rm -rf * || true'  
                    checkout scm
                    sh 'ls -la > $OUTPUT_FILE'
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                    echo "Verifying tools..." >> $OUTPUT_FILE
                    docker --version || echo "Docker not installed!" >> $OUTPUT_FILE
                    docker-compose --version || echo "Docker Compose not found!" >> $OUTPUT_FILE
                    sonar-scanner --version || echo "SonarScanner not installed!" >> $OUTPUT_FILE
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        echo "Running SonarQube analysis..." >> $OUTPUT_FILE
                        sonar-scanner \
                          -Dsonar.projectKey=${SONARQUBE_PROJECT_KEY} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${SONAR_HOST_URL} \
                          -Dsonar.login=$SONAR_TOKEN \
                          -Dsonar.qualitygate.wait=true \
                          -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**" >> $OUTPUT_FILE
                        '''
                    }
                }
            }
        }

        stage('Publish SonarQube Results') {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        waitForQualityGate abortPipeline: false
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . >> $OUTPUT_FILE || echo 'Docker build failed!' >> $OUTPUT_FILE"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_TOKEN')]) {
                        sh '''
                        echo "Logging into Docker Hub..." >> $OUTPUT_FILE
                        echo "$DOCKER_TOKEN" | docker login -u ahmedelshandidy --password-stdin
                        docker tag ${DOCKER_IMAGE}:latest ahmedelshandidy/${DOCKER_IMAGE}:latest
                        docker push ahmedelshandidy/${DOCKER_IMAGE}:latest >> $OUTPUT_FILE
                        '''
                    }
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    echo "Deploying application..." >> $OUTPUT_FILE
                    if [ -f docker-compose.yml ]; then
                        docker-compose down || echo "Failed to stop running containers" >> $OUTPUT_FILE
                        docker-compose up -d || echo "Failed to start containers" >> $OUTPUT_FILE
                    else
                        echo "⚠️ No docker-compose.yml found!" >> $OUTPUT_FILE
                    fi
                    '''
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    sh '''
                    echo "Running application tests..." >> $OUTPUT_FILE
                    if [ -f tests/run-tests.sh ]; then
                        chmod +x tests/run-tests.sh
                        ./tests/run-tests.sh >> $OUTPUT_FILE
                    else
                        echo "⚠️ No test script found!" >> $OUTPUT_FILE
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
            echo "❌ Deployment Failed. Check $OUTPUT_FILE!"
        }
    }
}
