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
                git branch: 'main', url: 'https://github.com/your-username/your-nopcommerce-repo.git'
            }
        }

        stage('Stop Existing Containers') {
            steps {
                sh 'docker-compose down || true'
            }
        }

        stage('Start Database') {
            steps {
                sh 'docker-compose up -d nopcommerce_database'
                echo 'Waiting for SQL Server to start...'
                sh 'sleep 20'  // Optional: wait for MSSQL to initialize
            }
        }

        stage('Database Backup') {
            steps {
                sh '''
                    mkdir -p $BACKUP_DIR
                    docker exec $DB_CONTAINER /opt/mssql-tools/bin/sqlcmd \
                      -S localhost -U sa -P $DB_PASSWORD \
                      -Q "BACKUP DATABASE [$DB_NAME] TO DISK = '/var/opt/mssql/backup/${DB_NAME}_jenkins.bak'"
                    
                    docker cp $DB_CONTAINER:/var/opt/mssql/backup/${DB_NAME}_jenkins.bak $BACKUP_DIR/
                '''
            }
        }

        stage('Build and Deploy App') {
            steps {
                sh 'docker-compose up -d --build'
            }
        }

        // Optional test stage if you want to run integration/unit tests
        stage('Test') {
            steps {
                echo 'Insert your test commands here (e.g., dotnet test or health check scripts)'
            }
        }
    }

    post {
        success {
            archiveArtifacts artifacts: "$BACKUP_DIR/*.bak", fingerprint: true
        }
        failure {
            mail to: 'your@email.com',
                 subject: "Jenkins Build Failed",
                 body: "The Jenkins build failed. Please check the console output for more info."
        }
    }
}

