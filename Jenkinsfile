pipeline {
    agent none 

    stages {
        // Stage 1: Checkout Code
        stage('Checkout Code') {
            agent { label 'maven-agent' } 
            steps {
                checkout scm
            }
        }

        // Stage 2: Build & Test
        stage('Build & Test') {
            agent { label 'maven-agent' }
            steps {
                echo 'Running Maven Test...'
                sh 'mvn clean test' 
            }
        }

        // Stage 3: Security Scan (Mock for Assignment)
        stage('Security Scan') {
             agent { label 'maven-agent' }
             steps {
                 echo 'Running Security Scan...'
                 // We simulate a scan here. Real tools like Trivy would go here.
                 sh 'echo "No high vulnerabilities found"' 
             }
        }

        // Stage 4: Package Artifact
        stage('Package') {
            agent { label 'maven-agent' }
            steps {
                echo 'Packaging Application...'
                // Skip tests here since we did them in Stage 2
                sh 'mvn package -DskipTests' 
                // Save the .jar file so Jenkins keeps it
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
            }
        }

        // Stage 5: Deploy (Only on 'main' branch)
        stage('Deploy to App Server') {
            when {
                branch 'master'
            }
            agent { label 'maven-agent' }
            steps {
                sshagent(['appserver']) { 
                    // Using your specific Private IP: 172.31.75.223
                    sh '''
                        scp -o StrictHostKeyChecking=no target/*.jar ubuntu@34.205.87.31:~/app.jar
                        ssh -o StrictHostKeyChecking=no ubuntu@34.205.87.31 "nohup java -jar ~/app.jar > output.log 2>&1 &"
                    ''' 
                }
            }
        }
    }

    // EXTRA TASKS: Notifications (Email & Slack)
    post {
        always {
            echo 'Build cycle complete.'
        }
        success {
            echo 'Build Succeeded! Sending notifications...'
            
            // 1. Email Notification (Requires SMTP setup in Manage Jenkins)
             mail to: 'kavilinikitha1999@gmail.com', subject: "SUCCESS: ${currentBuild.fullDisplayName}", body: "Build Passed!"
            
            // 2. Slack Notification (Requires Slack Plugin & Token)
            slackSend color: 'good', message: "SUCCESS: Build #${currentBuild.number} - ${currentBuild.fullDisplayName} (<${env.BUILD_URL}|Open>)"
        }
        failure {
            echo 'Build Failed! Sending alerts...'
            
            // 1. Email Notification
            mail to: 'kavilinikitha1999@gmail.com', subject: "FAILURE: ${currentBuild.fullDisplayName}", body: "Build Failed. Check logs."
            
            // 2. Slack Notification
            slackSend color: 'danger', message: "FAILED: Build #${currentBuild.number} - ${currentBuild.fullDisplayName} (<${env.BUILD_URL}|Open>)"
        }
    }
}
