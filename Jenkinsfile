pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "nopcommerce"
        DB_NAME = "nopcommerce2"
        DB_PASSWORD = "yourStrong()Password" // 🔒 Replace with Jenkins credentials or secrets in production
        DB_CONTAINER = "nopcommerce_mssql_server"
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

        stage('Backup Database') {
            steps {
                sh '''
                    echo "💾 Backing up database '${DB_NAME}'..."
                    BACKUP_DIR=/var/opt/mssql/backup
                    TIMESTAMP=$(date +%F_%H-%M-%S)
                    BACKUP_FILE="${BACKUP_DIR}/${DB_NAME}_${TIMESTAMP}.bak"

                    echo "📤 Running SQL Server backup inside container..."
                    docker exec -u 0 ${DB_CONTAINER} /opt/mssql-tools/bin/sqlcmd \
                        -S localhost -U sa -P "${DB_PASSWORD}" \
                        -Q "BACKUP DATABASE [${DB_NAME}] TO DISK='${BACKUP_FILE}'"

                    echo "📥 Copying backup file from container to workspace..."
                    mkdir -p backup
                    CONTAINER_BAK=$(docker exec ${DB_CONTAINER} find ${BACKUP_DIR} -type f -name "${DB_NAME}_*.bak" | tail -n1)

                    if [ -z "$CONTAINER_BAK" ]; then
                        echo "❌ No backup file found in container!"
                        exit 1
                    fi

                    echo "✅ Found backup file: $CONTAINER_BAK"
                    docker cp ${DB_CONTAINER}:"$CONTAINER_BAK" ./backup/
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
                    docker ps | grep -q ${DB_CONTAINER}

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
