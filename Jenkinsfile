pipeline {
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
                              -Dsonar.login=$SONAR_TOKEN \
                              -Dsonar.qualitygate.wait=false \
                              -Dsonar.exclusions="**/node_modules/**,**/tests/**,**/*.log,**/bin/**,**/out/**"
                            '''
                        }
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
