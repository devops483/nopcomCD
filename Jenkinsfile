pipeline {
    agent any

    environment {
        IMAGE_NAME = "nopcommerce_web"
        DB_CONTAINER = "nopcommerce_mssql_server"
        DB_PASSWORD = "yourStrong()Password"
        DB_NAME = "nopcommerce"
        BACKUP_DIR = "db_backups"
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

        stage('Wait for Manual Installation') {
            steps {
                input message: '''
                ✅ Please open your browser and finish installing nopCommerce at:
                http://<your-server-ip>:9000

                ⚠️ Once installation is complete and the nopcommerce database is created,
                click "Proceed" to back up the database.
                '''
            }
        }

        stage('Database Backup') {
            steps {
                sh '''
                    mkdir -p $BACKUP_DIR

                    docker exec $DB_CONTAINER mkdir -p /var/opt/mssql/backup || true

                    docker exec $DB_CONTAINER /opt/mssql-tools/bin/sqlcmd \
                      -S localhost -U sa -P $DB_PASSWORD \
                      -Q "BACKUP DATABASE [$DB_NAME] TO DISK = '/var/opt/mssql/backup/${DB_NAME}_jenkins.bak'"

                    docker cp $DB_CONTAINER:/var/opt/mssql/backup/${DB_NAME}_jenkins.bak $BACKUP_DIR/
                '''
            }
        }

        stage('Test') {
            steps {
                echo '✅ Insert test commands here (e.g., dotnet test, curl health check, etc.)'
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
