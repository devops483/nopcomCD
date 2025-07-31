pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "nopcommerce"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/devops483/nopcomCD.git'
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh '''
                    echo "🛑 Stopping and removing existing containers..."
                    docker-compose down || true
                    docker rm -f nopcommerce_mssql_server || true
                    docker rm -f nopcommerce_web || true
                '''
            }
        }

        stage('Start All Containers') {
            steps {
                sh '''
                    echo "🚀 Starting Docker containers..."
                    docker-compose up -d

                    echo "⏳ Waiting for services to be ready..."
                    sleep 30
                '''
            }
        }

        stage('Smoke Tests') {
            steps {
                sh '''
                    set -e

                    echo "🔍 Checking web container is up..."
                    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9000)
                    [ "$HTTP_CODE" = "200" ]

                    echo "🔍 Checking nopCommerce keyword on homepage..."
                    curl -vsSf http://localhost:9000 | grep -qi "nopCommerce"

                    echo "🔍 Checking login page content..."
                    curl -vsSf http://localhost:9000/login | grep -qi "Email"

                    echo "🔍 Ensuring DB container is running..."
                    docker ps | grep -q nopcommerce_mssql_server

                    echo "✅ All smoke tests passed."
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully."
        }
        failure {
            echo "❌ Build failed. Please check which step failed."
        }
    }
}
