pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "the-final-project-guestbook-web"
        DOCKER_HUB_USERNAME = "ahmedelshandidy"
        OUTPUT_LOG = "pipeline_output.log"
    }

    stages {
        stage('Cleanup Old Images') {
            steps {
                script {
                    sh '''
                    echo "Starting Cleanup..." > $OUTPUT_LOG
                    docker image prune -af --filter "label!=jenkins"
                    '''
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
                    ansible --version || echo "Ansible not installed!" | tee -a $OUTPUT_LOG
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:latest . | tee -a $OUTPUT_LOG"
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
                    sh 'ansible-playbook deploy.yml | tee -a $OUTPUT_LOG'
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
