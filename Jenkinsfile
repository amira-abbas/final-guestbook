pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        DOCKER_HUB_USERNAME = "ahmedelshandidy"
        OUTPUT_LOG = "pipeline_output.log"
        SLACK_WEBHOOK_URL = "https://hooks.slack.com/services/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"  // Slack Webhook URL
    }

    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh 'echo "Starting Cleanup..." > $OUTPUT_LOG'
                    sh 'rm -rf * || true'  
                    sh 'rm -f $OUTPUT_LOG || true'  
                }
            }
        }

        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                    sh 'ls -la | tee -a $OUTPUT_LOG'
                }
            }
        }

        stage('Verify Environment') {
            steps {
                script {
                    sh '''
                    docker --version || echo "Docker not installed!" | tee -a $OUTPUT_LOG
                    docker-compose --version || echo "Docker Compose not found!" | tee -a $OUTPUT_LOG
                    sonar-scanner --version || echo "SonarScanner not installed!" | tee -a $OUTPUT_LOG
                    '''
                }
            }
        }

        stage('Ensure SonarQube is Running') {
            steps {
                script {
                    def sonarStatus = sh(script: "docker inspect -f '{{.State.Running}}' sonarqube || echo 'false'", returnStdout: true).trim()
                    if (sonarStatus != 'true') {
                        echo "🚀 SonarQube is not running. Starting it now..."
                        sh '''
                        docker start sonarqube || \
                        docker run -d --name sonarqube --restart always -p 9000:9000 sonarqube:lts
                        sleep 30  # Wait for SonarQube to start
                        '''
                    } else {
                        echo "✅ SonarQube is already running."
                    }
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
                          | tee -a $OUTPUT_LOG
                        '''
                    }
                }
            }
        }

        stage('Publish SonarQube Report') {
            steps {
                script {
                    sh 'cp .scannerwork/report-task.txt sonar-report.log || echo "No SonarQube report found!" | tee -a $OUTPUT_LOG'
                }
                archiveArtifacts artifacts: 'sonar-report.log', fingerprint: true
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . | tee -a $OUTPUT_LOG || echo 'Docker build failed!' | tee -a $OUTPUT_LOG"
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
                    if [ -f docker-compose.yml ]; then
                        docker-compose down || echo "Failed to stop running containers" | tee -a $OUTPUT_LOG
                        docker-compose up -d || echo "Failed to start containers" | tee -a $OUTPUT_LOG
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
        always {
            archiveArtifacts artifacts: 'pipeline_output.log', fingerprint: true
        }
        success {
            echo "✅ Deployment Successful!"
            script {
                // Send success message to Slack
                sh '''
                curl -X POST --data-urlencode "payload={\"channel\": \"#jenkins\", \"text\": \"✅ Deployment Successful!\"}" "${SLACK_WEBHOOK_URL}"
                '''
            }
        }
        failure {
            echo "❌ Deployment Failed. Check logs!"
            script {
                // Send failure message to Slack
                sh '''
                curl -X POST --data-urlencode "payload={\"channel\": \"#jenkins\", \"text\": \"❌ Deployment Failed! Check logs.\"}" "${SLACK_WEBHOOK_URL}"
                '''
            }
        }
    }
}
