pipeline {
    agent any

    environment {
        IMAGE_NAME = "nopcommerce_web"
        DB_CONTAINER = "nopcommerce_mssql_server"
        DB_PASSWORD = "yourStrong()Password"
        DB_NAME = "nopcommerce"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/devops483/nopcomCD.git'
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh '''
                    docker-compose down || true
                    docker rm -f nopcommerce_mssql_server || true
                    docker rm -f nopcommerce_web || true
                '''
            }
        }

        stage('Start All Containers') {
            steps {
                sh '''
                    docker-compose up -d
                    echo "Waiting for services to be ready..."
                    sleep 30
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    echo "🔍 Checking web container is up..."
                    curl -s -o /dev/null -w "%{http_code}\\n" http://localhost:9000 | grep 200

                    echo "🔍 Checking nopCommerce keyword on homepage..."
                    curl -s http://localhost:9000 | grep -qi "nopCommerce"

                    echo "🔍 Checking login page content..."
                    curl -s http://localhost:9000/login | grep -qi "Email"

                    echo "🔍 Ensuring DB container is running..."
                    docker ps | grep -q nopcommerce_mssql_server

                    echo "✅ All basic tests passed."
                '''
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed. Email notification skipped (SMTP not configured).'
        }
        success {
            echo '✅ Build and deployment complete.'
        }
    }
}
