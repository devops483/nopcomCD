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
                    set -e  # Stop on any command failure

                    echo "🔍 Checking web container is up..."
                    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9000)
                    [ "$HTTP_CODE" = "200" ] || { echo "❌ Web container not responding properly."; exit 1; }

                    echo "🔍 Checking nopCommerce keyword on homepage..."
                    curl -s http://localhost:9000 | grep -qi "nopCommerce" || { echo "❌ nopCommerce keyword not found on homepage."; exit 1; }

                    echo "🔍 Checking login page content..."
                    curl -s http://localhost:9000/login | grep -qi "Email" || { echo "❌ Login page does not contain expected content."; exit 1; }

                    echo "🔍 Ensuring DB container is running..."
                    docker ps | grep -q nopcommerce_mssql_server || { echo "❌ Database container is not running."; exit 1; }

                    echo "✅ All basic tests passed."
                '''
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed. Please check which test or step failed.'
        }
        success {
            echo '✅ Build and deployment complete. All checks passed.'
        }
    }
}
