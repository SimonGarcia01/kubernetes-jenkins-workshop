pipeline {
    agent any
    environment {
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONAR_PROJECT_KEY = 'my-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'chmod +x mvnw'
                sh './mvnw -B clean package'
            }
        }

        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    def loginArg = env.SONAR_TOKEN ? "-Dsonar.login=${env.SONAR_TOKEN}" : ''
                    sh "./mvnw -B sonar:sonar -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} -Dsonar.host.url=${env.SONAR_HOST_URL} ${loginArg}"
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'for i in $(seq 1 20); do docker version && break || sleep 2; done'
                sh 'docker build -t mi-app:latest .'
            }
        }

        stage('Container Security Scan (Trivy)') {
            steps {
                sh 'docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.50.2 image mi-app:latest'
            }
        }

        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh 'docker run -d -p 8080:8080 mi-app:latest'
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
            echo 'Limpiando entorno...'
            script {
                try {
                    cleanWs()
                } catch (err) {
                    echo 'cleanWs no esta disponible'
                }
            }
        }
    }
}