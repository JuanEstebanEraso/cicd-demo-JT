pipeline {
    agent any
    environment {
        SONAR_PROJECT_KEY = 'mi-app'
        SONAR_HOST_URL = 'http://sonarqube:9000'
        SONAR_TOKEN = credentials('sonar-token') // Configura este secreto en Jenkins
        TRIVY_SEVERITY = 'CRITICAL'
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Static Analysis (SonarQube)') {
            steps {
                withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
                    sh "mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY} -Dsonar.host.url=${SONAR_HOST_URL} -Dsonar.login=${SONAR_TOKEN}"
                }
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t mi-app:latest .'
            }
        }
        stage('Container Security Scan (Trivy)') {
            steps {
                sh 'trivy image --exit-code 1 --severity ${TRIVY_SEVERITY} mi-app:latest'
            }
        }
        stage('Deploy') {
            when { branch 'main' }
            steps {
                sh 'docker rm -f mi-app-container || true'
                sh 'docker run -d --name mi-app-container -p 80:8080 mi-app:latest'
            }
        }
    }
    post {
        always {
            echo 'Limpiando entorno...'
            cleanWs()
        }
        failure {
            echo 'El pipeline ha fallado. Revisa los logs.'
        }
    }
}
