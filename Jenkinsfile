pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "the-final-project-guestbook-web"
        DOCKER_HUB_USERNAME = "ahmedelshandidy"
        SONARQUBE_SERVER = "SonarQube"
        SONARQUBE_PROJECT_KEY = "final-guestbook"
        SONAR_HOST_URL = "http://16.170.182.27:9000"
        OUTPUT_LOG = "pipeline_output.log"
    }

    stages {
        stage('Cleanup') {
            steps {
                script {
                    sh 'echo "Starting Cleanup..." > $OUTPUT_LOG'
                    sh 'docker stop $(docker ps -q) || true'
                    sh 'docker system prune -af || true'
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

        stage('Deploy with Ansible') {
            steps {
                script {
                    sh '''
                    ansible-playbook -i inventory deploy_final_project.yml | tee -a $OUTPUT_LOG
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
        }
        failure {
            echo "❌ Deployment Failed. Check logs!"
        }
    }
}
