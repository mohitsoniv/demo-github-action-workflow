pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "Stopping Tomcat..."
                sudo /opt/tomcat/bin/shutdown.sh || true

                echo "Removing old app..."
                sudo rm -rf /opt/tomcat/webapps/insured-assurance

                echo "Deploying new WAR..."
                sudo cp target/insured-assurance.war /opt/tomcat/webapps/

                echo "Starting Tomcat..."
                sudo /opt/tomcat/bin/startup.sh
                '''
            }
        }
    }
}
