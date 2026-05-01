pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -B -DskipTests package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t mi-app:latest .'
            }
        }

        stage('Test') {
            steps {
                sh './mvnw -B test'
            }
        }
    }

    post {
        always {
            script {
                def hasReports = sh(
                    script: 'ls -1 target/surefire-reports/*.xml >/dev/null 2>&1',
                    returnStatus: true
                ) == 0
                if (hasReports) {
                    junit 'target/surefire-reports/*.xml'
                }
            }
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, allowEmptyArchive: true
        }
    }
}