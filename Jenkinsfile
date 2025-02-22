pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // Add commands to build your project here
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                // Add commands to test your project here
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying...'
                // Add commands to deploy your project here
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            // Cleanup commands here
        }
    }
}

