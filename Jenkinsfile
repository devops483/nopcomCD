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

        stage('Basic Smoke Tests') {
            steps {
                sh '''
                    set -euo pipefail

                    echo "🔍 Checking web container is up..."
                    for i in {1..5}; do
                      HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:9000 || echo "000")
                      if [ "$HTTP_CODE" = "200" ]; then
                        echo "✅ Web container is responsive."
                        break
                      fi
                      echo "⏳ Attempt $i failed with code $HTTP_CODE. Retrying in 5s..."
                      sleep 5
                    done

                    [ "$HTTP_CODE" = "200" ] || { echo "❌ Web container failed to respond with HTTP 200."; exit 1; }

                    echo "🔍 Verifying homepage content..."
                    curl -s http://localhost:9000 | grep -qi "nopCommerce" || {
                        echo "❌ 'nopCommerce' keyword not found on homepage."; exit 1;
                    }

                    echo "🔍 Verifying login page content..."
                    curl -s http://localhost:9000/login | grep -qi "Email" || {
                        echo "❌ Login page does not contain expected content."; exit 1;
                    }

                    echo "🔍 Ensuring DB container is running..."
                    docker ps | grep -q "$DB_CONTAINER" || {
                        echo "❌ Database container '$DB_CONTAINER' is not running."; exit 1;
                    }

                    echo "✅ All smoke tests passed."
                '''
            }
        }

        stage('Run Unit Tests in Container') {
            agent {
                docker {
                    image 'mcr.microsoft.com/dotnet/sdk:8.0'
                    args '-v $HOME/.nuget/packages:/root/.nuget/packages' // Optional cache reuse
                }
            }
            steps {
                sh '''
                    echo "🧪 Restoring, building, and testing .NET solution..."

                    dotnet restore
                    dotnet build --no-restore
                    dotnet test --no-build --logger "trx;LogFileName=test_results.trx"

                    echo "✅ Unit tests completed."
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
