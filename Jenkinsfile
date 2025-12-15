pipeline {
    agent none

    stages {

        // -------------------------------
        // Stage 1: Checkout Code
        // -------------------------------
        stage('Checkout Code') {
            steps {
                node('maven-agent') {
                checkout scm
            }
        }

        // -------------------------------
        // Stage 2: Build & Test
        // -------------------------------
        stage('Build & Test') {
            steps {
                node('maven-agent){
                sh 'mvn clean test'
            }
        }

        // -------------------------------
        // Stage 3: Security Scan (Trivy)
        // -------------------------------
        stage('Security Scan') {
            agent { label 'maven-agent' }
            steps {
                echo 'Running Trivy security scan...'
                // Scan project filesystem for vulnerabilities
                sh '''
                    trivy fs --exit-code 0 --severity HIGH,CRITICAL .
                '''
            }
        }

        // -------------------------------
        // Stage 4: Package Artifact
        // -------------------------------
        stage('Package') {
            agent { label 'maven-agent' }
            steps {
                echo 'Packaging application...'
                sh 'mvn package -DskipTests'

                // Archive generated JAR file
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }

        // -------------------------------
        // Stage 5: Deploy (Only on master)
        // -------------------------------
        stage('Deploy to App Server') {
            when {
                branch 'master'
            }
            agent { label 'maven-agent' }
            steps {
                echo 'Deploying application to EC2 server...'

                // Uses SSH private key stored in Jenkins credentials (ID: appserver)
                sshagent(['appserver']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no target/*.jar ubuntu@34.205.87.31:~/app.jar
                        ssh -o StrictHostKeyChecking=no ubuntu@34.205.87.31 \
                        "nohup java -jar ~/app.jar > app.log 2>&1 &"
                    '''
                }
            }
        }
    }

    // -------------------------------
    // Post Build Actions
    // -------------------------------
    post {
        always {
            echo 'Build cycle complete.'
        }

        success {
            echo 'Build succeeded.'

            // Email notification (requires SMTP configured)
            mail to: 'kavilinikitha1999@gmail.com',
                 subject: "SUCCESS: ${currentBuild.fullDisplayName}",
                 body: "Build completed successfully.\n\n${env.BUILD_URL}"

            // Slack notification (requires Slack plugin + token)
            slackSend color: 'good',
                      message: "SUCCESS: Build #${currentBuild.number} - ${currentBuild.fullDisplayName}\n${env.BUILD_URL}"
        }

        failure {
            echo 'Build failed.'

            // Email notification
            mail to: 'kavilinikitha1999@gmail.com',
                 subject: "FAILURE: ${currentBuild.fullDisplayName}",
                 body: "Build failed.\nCheck logs: ${env.BUILD_URL}"

            // Slack notification
            slackSend color: 'danger',
                      message: "FAILED: Build #${currentBuild.number} - ${currentBuild.fullDisplayName}\n${env.BUILD_URL}"
        }
    }
}
