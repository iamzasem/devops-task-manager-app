pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'docker compose build'
            }
        }

        stage('Deploy') {
            steps {
                sh 'cp /opt/devops-task-manager/.env .env'
                sh 'docker compose up -d --remove-orphans'
            }
        }

        stage('Verify') {
            steps {
                sh 'docker compose ps'
                sh 'curl -f http://localhost/api/health'
            }
        }
    }
}
