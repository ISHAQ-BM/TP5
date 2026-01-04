pipeline {
    agent any

    tools {
        jdk 'java11' // adjust to your Jenkins JDK name
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                sh './gradlew test cucumber'
                junit '**/build/test-results/test/*.xml'
            }
        }

        stage('Code Analysis') {
            steps {
                sh './gradlew sonarqube'
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Quality Gate failed"
                    }
                }
            }
        }

        stage('Build') {
            steps {
                sh './gradlew build javadoc'
                archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
            }
        }

        stage('Deploy') {
            steps {
                sh './gradlew publish'
            }
        }
    }
}
