pipeline {
    agent any

    environment {
        APP_NAME = "insured-assurance"
        TOMCAT_PATH = "/opt/tomcat/webapps"
        BACKUP_PATH = "/opt/tomcat/backup"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."
                sh 'mvn clean package'
            }
        }

        stage('Backup Current Version') {
            steps {
                sh '''
                echo "Creating backup directory..."
                sudo mkdir -p $BACKUP_PATH

                if [ -f "$TOMCAT_PATH/$APP_NAME.war" ]; then
                    echo "Backing up existing WAR..."
                    sudo cp $TOMCAT_PATH/$APP_NAME.war $BACKUP_PATH/$APP_NAME-$(date +%F-%H%M%S).war
                else
                    echo "No existing WAR found"
                fi
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "Stopping Tomcat..."
                sudo /opt/tomcat/bin/shutdown.sh || true

                echo "Deploying new WAR..."
                sudo cp target/$APP_NAME.war $TOMCAT_PATH/

                echo "Starting Tomcat..."
                sudo /opt/tomcat/bin/startup.sh
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Checking application health..."
                sleep 15

                curl -f http://localhost:8080/$APP_NAME || exit 1
                '''
            }
        }
    }

    post {

        failure {
            echo "Deployment failed. Rolling back..."

            sh '''
            echo "Stopping Tomcat..."
            sudo /opt/tomcat/bin/shutdown.sh || true

            LAST_BACKUP=$(ls -t $BACKUP_PATH/$APP_NAME-*.war | head -1)

            if [ -f "$LAST_BACKUP" ]; then
                echo "Restoring backup: $LAST_BACKUP"
                sudo cp $LAST_BACKUP $TOMCAT_PATH/$APP_NAME.war
            else
                echo "No backup found. Rollback skipped!"
            fi

            echo "Starting Tomcat..."
            sudo /opt/tomcat/bin/startup.sh
            '''
        }

        success {
            echo "Deployment successful 🎉"
        }
    }
}
