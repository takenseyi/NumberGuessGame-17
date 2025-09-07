pipeline {
    agent any

    tools {
        maven 'Maven_3'
    }

    environment {
        NEXUS_URL = "http://35.173.124.242:8081/nexus/"
        NEXUS_CREDENTIALS_ID = "nexus"
        ARTIFACT_NAME = "NumberGuessGame"
        GROUP_ID = "com.studentapp"
        VERSION = "1.0.${BUILD_NUMBER}"
        SONARQUBE_SERVER = "My SonarQube Server"
        SONAR_HOST_URL = "http://54.205.217.253:9000"
        TOMCAT_SSH_CREDENTIALS = "ec2batekey"
        TOMCAT_HOST = "ec2-user@35.173.219.9"
        TOMCAT_DEPLOY_DIR = "~/apache-tomcat-7.0.94/webapps"
        GIT_REPO = "https://github.com/takenseyi/NumberGuessGame-17.git"
        GIT_CREDENTIALS_ID = "github-token"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: "${GIT_REPO}"
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
                            -Dsonar.host.url=${SONAR_HOST_URL} \\
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
                        "http://35.173.124.242/nexus/content/repositories/releases/com/studentapp/${ARTIFACT_NAME}/${VERSION}/${ARTIFACT_NAME}-${VERSION}.war" \\
                        -O ${TOMCAT_DEPLOY_DIR}/${ARTIFACT_NAME}-${VERSION}.war && \\
                        ${TOMCAT_DEPLOY_DIR}/../bin/shutdown.sh && \\
                        sleep 5 && \\
                        ${TOMCAT_DEPLOY_DIR}/../bin/startup.sh'
                    """
                }
            }
        }

        stage('Tag Git Version') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh """
                        git config user.name "jenkins"
                        git config user.email "jenkins@local"
                        git tag -a v${VERSION} -m "Deployed version ${VERSION}"
                        git push https://${GIT_USER}:${GIT_PASS}@github.com/takenseyi/NumberGuessGame-17.git v${VERSION}
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
            echo ":white_check_mark: Build ${VERSION} deployed and tagged successfully!"
        }
        failure {
            echo ":x: Build failed. Check logs for details."
        }
    }
}


