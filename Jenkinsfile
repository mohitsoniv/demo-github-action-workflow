pipeline {
    agent any

    environment {
        APP_NAME = "insured-assurance"
        TOMCAT_HOME = "/opt/tomcat"
        BACKUP_DIR = "/opt/tomcat/backup"
        HEALTH_URL = "http://localhost:8080/insured-assurance/index.jsp"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/mohitsoniv/demo-github-action-workflow.git'
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
                sudo mkdir -p $BACKUP_DIR

                if [ -f $TOMCAT_HOME/webapps/$APP_NAME.war ]; then
                    echo "Backing up existing WAR..."
                    TIMESTAMP=$(date +%F-%H%M%S)
                    sudo cp $TOMCAT_HOME/webapps/$APP_NAME.war \
                    $BACKUP_DIR/$APP_NAME-$TIMESTAMP.war
                else
                    echo "No existing WAR found. Skipping backup."
                fi
                '''
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "Stopping Tomcat..."
                sudo $TOMCAT_HOME/bin/shutdown.sh || true
                sleep 5

                echo "Deploying new WAR..."
                sudo cp target/$APP_NAME.war $TOMCAT_HOME/webapps/

                echo "Starting Tomcat..."
                sudo $TOMCAT_HOME/bin/startup.sh
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                echo "Waiting for app to start..."
                sleep 20

                echo "Running health check..."
                curl -f $HEALTH_URL
                '''
            }
        }
    }

    post {

        success {
            echo "Deployment successful 🎉"
        }

        failure {
            echo "Deployment failed. Rolling back..."

            sh '''
            echo "Stopping Tomcat..."
            sudo $TOMCAT_HOME/bin/shutdown.sh || true
            sleep 5

            LAST_BACKUP=$(ls -t $BACKUP_DIR/$APP_NAME-*.war | head -1)

            if [ -f "$LAST_BACKUP" ]; then
                echo "Restoring backup: $LAST_BACKUP"
                sudo cp $LAST_BACKUP \
                $TOMCAT_HOME/webapps/$APP_NAME.war
            else
                echo "No backup found! Manual recovery required."
                exit 1
            fi

            echo "Starting Tomcat..."
            sudo $TOMCAT_HOME/bin/startup.sh
            '''
        }
    }
}
