pipeline {
    agent any

    tools {
        maven 'Maven_3' // Matches your configured Maven tool name
    }

    environment {
        DEPLOY_DIR = "/var/lib/tomcat9/webapps"
        NEXUS_URL = "http://44.202.160.205:8081/repository/maven-releases/"
        NEXUS_CREDENTIALS_ID = "nexus" // Set this in Jenkins credentials
        ARTIFACT_NAME = "NumberGuessGame"
        VERSION = "1.0.0"
        GROUP_ID = "com/example"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/takenseyi/NumberGuessGame-17.git'
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

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                    echo "Uploading artifact to Nexus..."
                    curl -v -u $NEXUS_USER:$NEXUS_PASS \
                    --upload-file target/${ARTIFACT_NAME}.war \
                    ${NEXUS_URL}${GROUP_ID}/${ARTIFACT_NAME}/${VERSION}/${ARTIFACT_NAME}-${VERSION}.war
                    '''
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                sh '''
                echo "Deploying WAR file to Tomcat..."
                cp target/${ARTIFACT_NAME}.war ${DEPLOY_DIR}/${ARTIFACT_NAME}.war
                '''
            }
        }
    }

    post {
        success {
            echo '✅ CI/CD pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check logs for details.'
        }
    }
}
