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

        stage('Smoke Tests') {
            steps {
                sh '''
                    set -e  # Stop on failure

                    echo "🔍 Checking web container is up..."
                    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9000)
                    [ "$HTTP_CODE" = "200" ] || { echo "❌ Web container not responding properly."; exit 1; }

                    echo "🔍 Checking nopCommerce keyword on homepage..."
                    curl -s http://localhost:9000 | grep -qi "nopCommerce" || { echo "❌ nopCommerce keyword not found on homepage."; exit 1; }

                    echo "🔍 Checking login page content..."
                    curl -s http://localhost:9000/login | grep -qi "Email" || { echo "❌ Login page does not contain expected content."; exit 1; }

                    echo "🔍 Ensuring DB container is running..."
                    docker ps | grep -q nopcommerce_mssql_server || { echo "❌ Database container is not running."; exit 1; }

                    echo "✅ All smoke tests passed."
                '''
            }
        }

        stage('Run Unit Tests in Container') {
            steps {
                sh '''
                    echo "🧪 Running unit tests inside .NET SDK container..."

                    docker run --rm -v "$PWD:/app" -w /app \
                        mcr.microsoft.com/dotnet/sdk:8.0 \
                        sh -c "dotnet restore && dotnet build --no-restore && dotnet test --no-build --logger 'trx;LogFileName=test_results.trx'"

                    echo "✅ Unit tests completed."
                '''
            }
        }
    }

    post {
        failure {
            echo '❌ Build failed. Please check which step failed.'
        }
        success {
            echo '✅ Build and deployment complete. All tests passed.'
        }
    }
}
