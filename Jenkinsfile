pipeline {
    agent { label = 'Built-In Node' }


    environment {
        APP_NAME = "my-app"
        ARTIFACT = "**/*.jar"

        DEPLOY_USER = "ubuntu"
        DEPLOY_HOST = "34.205.87.31"
        DEPLOY_PATH = "/home/ubuntu"

        SSH_CREDENTIALS = "appserver"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh '''

                  mvn clean test -DskipTests
                '''
            }
        }

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

        stage('Package Application') {
            steps {
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

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
