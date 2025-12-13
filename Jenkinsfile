pipeline {
    agent any

    tools {
        maven 'Maven-3.9.11'
    }

    environment {
        APP_NAME = "my-app"
        ARTIFACT = "**/*.jar"

        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "10.10.1.126"
        DEPLOY_PATH = "/opt/apps"

        SSH_CREDENTIALS = "appserver"
    }

    stages {

        /* 1. Checkout Code */
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        /* 2. Build & Test using Maven */
        stage('Build & Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        /* 3. Security Scan Stage */
        stage('Security Scan') {
            steps {
                sh '''
                  trivy fs \
                  --exit-code 1 \
                  --severity HIGH,CRITICAL \
                  .
                '''
            }
        }

        /* 4. Package Stage */
        stage('Package Application') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        /* 5. Deploy to Application Server (main branch only) */
        stage('Deploy to Application Server') {
            when {
                branch 'main'
            }
            steps {
                sshagent(credentials: ["${SSH_CREDENTIALS}"]) {
                    sh '''
                      scp target/*.jar ${DEPLOY_USER}@${DEPLOY_HOST}:${DEPLOY_PATH}/
                      ssh ${DEPLOY_USER}@${DEPLOY_HOST} "sudo systemctl restart ${APP_NAME}"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed – check logs"
        }
    }
}
