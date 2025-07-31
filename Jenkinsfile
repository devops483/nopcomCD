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
                sh 'docker-compose down || true'
            }
        }

        stage('Start All Containers') {
            steps {
                sh 'docker-compose up -d'
                echo 'Waiting for nopCommerce and SQL Server to start...'
                sh 'sleep 30'
            }
        }

        stage('Wait for Manual Installation') {
            steps {
                input message: 'Please complete nopCommerce installation in the browser, then click Continue to proceed with backup.'
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
                echo 'Insert your test commands here (e.g., dotnet test or health check scripts)'
            }
        }
    }

    post {
        failure {
            echo 'Build failed. Email notification skipped (SMTP not configured).'
        }
    }
}
