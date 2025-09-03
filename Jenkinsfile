pipeline {
    agent any

    tools {
        maven 'Maven 3.8.5' // Make sure this matches the Maven name in Jenkins tool config
    }

    environment {
        DEPLOY_DIR = "/var/lib/tomcat9/webapps" // Adjust if your Tomcat is in a different location
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Efosa234/NumberGuessGame.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                echo "Deploying WAR file to Tomcat..."
                cp target/*.war $DEPLOY_DIR/NumberGuessGame.war
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment successful!'
        }
        failure {
            echo '❌ Build or Deployment failed.'
        }
    }
}
