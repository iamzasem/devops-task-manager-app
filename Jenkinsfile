pipeline {
    agent any

    options {
        timestamps()
    }

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
                sh '''
                    set -eu

                    echo "Waiting for backend to become healthy..."

                    for attempt in $(seq 1 20); do
                        if curl -fsS --max-time 5 \
                            http://127.0.0.1:5000/api/health \
                            -o /tmp/backend-health.json; then
                            echo "Backend health check passed."
                            cat /tmp/backend-health.json
                            break
                        fi

                        if [ "$attempt" -eq 20 ]; then
                            echo "Backend did not become healthy."
                            docker compose logs --tail=80 backend
                            exit 1
                        fi

                        echo "Backend not ready. Retry ${attempt}/20..."
                        sleep 3
                    done

                    echo "Checking application through Nginx..."

                    for attempt in $(seq 1 20); do
                        if curl -fsS --max-time 5 \
                            http://devops.local/api/health \
                            -o /tmp/nginx-health.json; then
                            echo "Nginx health check passed."
                            cat /tmp/nginx-health.json
                            exit 0
                        fi

                        if [ "$attempt" -eq 20 ]; then
                            echo "Nginx health check failed."
                            exit 1
                        fi

                        echo "Nginx not ready. Retry ${attempt}/20..."
                        sleep 3
                    done
                '''
            }
        }
    }
}