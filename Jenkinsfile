
pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Environment') {
            steps {
                bat 'java -version'
                bat 'mvn -version'
            }
        }

        stage('Clean and Test') {
            steps {
                bat 'mvn -B clean test'
            }
        }

        stage('Package Application') {
            steps {
                bat 'mvn -B package'
            }
        }
    }

    post {
        always {
            junit testResults: 'course-registration-app/target/surefire-reports/TEST-*.xml',
                  allowEmptyResults: true
        }

        success {
            archiveArtifacts artifacts: 'course-registration-app/target/*.jar',
                             fingerprint: true
        }
    }
}
