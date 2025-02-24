pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "final-guestbook"
        DOCKER_HUB_USERNAME = "ahmedelshandidy"
        OUTPUT_LOG = "pipeline_output.log"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    checkout scm
                    sh 'ls -la >> $OUTPUT_LOG'
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-hub-token', variable: 'DOCKER_HUB_TOKEN')]) {
                        sh '''
                        docker build -t ${DOCKER_IMAGE}:latest .
                        echo "$DOCKER_HUB_TOKEN" | docker login -u "$DOCKER_HUB_USERNAME" --password-stdin
                        docker tag ${DOCKER_IMAGE}:latest $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest
                        docker push $DOCKER_HUB_USERNAME/${DOCKER_IMAGE}:latest
                        '''
                    }
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                script {
                    sh '''
                    ansible-playbook -i inventory deploy_final_project.yml -e "project_path=$WORKSPACE" >> $OUTPUT_LOG || echo "❌ Ansible deployment failed!" >> $OUTPUT_LOG
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
