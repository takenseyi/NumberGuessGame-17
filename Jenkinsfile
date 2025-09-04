pipeline {
    agent any

    tools {
        maven 'Maven_3'
    }

    environment {
        NEXUS_URL = "http://44.201.144.123:8081/nexus/"
        NEXUS_CREDENTIALS_ID = "nexus"
        ARTIFACT_NAME = "NumberGuessGame"
        VERSION = "1.0.0"
        GROUP_ID = "com.studentapp"
        SONARQUBE_SERVER = "My SonarQube Server"
        SONAR_HOST_URL = "http://44.203.69.191:9000"
        TOMCAT_SSH_CREDENTIALS = "ec2batekey"
        TOMCAT_HOST = "ec2-user@13.221.37.192"
        TOMCAT_DEPLOY_DIR = "~/apache-tomcat-7.0.94/webapps"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/takenseyi/NumberGuessGame-17.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    sh """
                        mvn sonar:sonar \\
                            -Dsonar.projectKey=NumberGuessGame \\
                            -Dsonar.projectName=NumberGuessGame \\
                            -Dsonar.host.url=$SONAR_HOST_URL \\
                            -Dsonar.sources=src/main/java \\
                            -Dsonar.tests=src/test/java
                    """
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${NEXUS_CREDENTIALS_ID}", usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh """
                        echo "Uploading artifact to Nexus..."
                        curl -v -u $NEXUS_USER:$NEXUS_PASS \\
                        ${NEXUS_URL}service/local/artifact/maven/content \\
                        -F r=releases \\
                        -F g=${GROUP_ID} \\
                        -F a=${ARTIFACT_NAME} \\
                        -F v=${VERSION} \\
                        -F p=war \\
                        -F e=war \\
                        -F file=@target/${ARTIFACT_NAME}-1.0-SNAPSHOT.war
                    """
                }
            }
        }

        stage('Deploy to Remote Tomcat') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${TOMCAT_SSH_CREDENTIALS}", keyFileVariable: 'KEY')]) {
                    sh """
                        echo "Deploying WAR file to remote Tomcat server via wget..."
                        ssh -o StrictHostKeyChecking=no -i $KEY ${TOMCAT_HOST} \\
                        'wget --http-user=admin --http-password=admin123 \\
                        "http://44.201.144.123:8081/nexus/content/repositories/releases/com/studentapp/NumberGuessGame/1.0.0/NumberGuessGame-1.0.0.war" \\
                        -O ${TOMCAT_DEPLOY_DIR}/${ARTIFACT_NAME}.war'
                    """
                }
            }
        }
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'
        }
        success {
            echo ':white_check_mark: CI/CD pipeline completed successfully!'
        }
        failure {
            echo ':x: Pipeline failed. Check logs for details.'
        }
    }
}

