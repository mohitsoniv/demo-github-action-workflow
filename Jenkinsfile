pipeline {
    agent any

    tools {
        jdk 'jdk-21'
        maven 'maven-3'
    }

    environment {
        TOMCAT_HOME = "/opt/tomcat"
        APP_NAME = "insured-assurance"
    }

    stages {
        stage('Verify Java') {
            steps {
                sh 'java -version'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                $TOMCAT_HOME/bin/shutdown.sh || true
                rm -rf $TOMCAT_HOME/webapps/$APP_NAME
                cp target/$APP_NAME.war $TOMCAT_HOME/webapps/
                $TOMCAT_HOME/bin/startup.sh
                '''
            }
        }
    }
}