pipeline {
    agent any

    tools {
        maven 'Maven_3'
    }

    environment {
        NEXUS_URL = "http://3.93.23.105:8081/nexus/"
        NEXUS_CREDENTIALS_ID = "nexus"
        ARTIFACT_NAME = "NumberGuessGame"
        GROUP_ID = "com.studentapp"
        VERSION = "1.0.${BUILD_NUMBER}"
        SONARQUBE_SERVER = "My SonarQube Server"
        SONAR_HOST_URL = "http://44.211.82.116:9000"
        TOMCAT_SSH_CREDENTIALS = "ec2batekey"
        TOMCAT_HOST = "ec2-user@34.207.139.120"
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
                sh "mvn clean package -Dproject.version=${VERSION}"
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
                        -F file=@target/${ARTIFACT_NAME}-${VERSION}.war
                    """
                }
            }
        }

        stage('Deploy to Remote Tomcat') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: "${TOMCAT_SSH_CREDENTIALS}", keyFileVariable: 'KEY')]) {
                    sh """
                        echo "Deploying WAR file to remote Tomcat server via wget..."
                        ssh -o StrictHostKeyChecking=no -i "$KEY" ${TOMCAT_HOST} \\
                        'rm -f ${TOMCAT_DEPLOY_DIR}/${ARTIFACT_NAME}*.war && \\
                        wget --http-user=admin --http-password=admin123 \\
                        "http://3.93.23.105:8081/nexus/content/repositories/releases/com/studentapp/${ARTIFACT_NAME}/${VERSION}/${ARTIFACT_NAME}-${VERSION}.war" \\
                        -O ${TOMCAT_DEPLOY_DIR}/${ARTIFACT_NAME}-${VERSION
