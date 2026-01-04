pipeline {
    agent any

    tools {
        jdk 'Java11'  // adjust your Jenkins JDK name
        gradle 'Gradle' // adjust Gradle installation name in Jenkins
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
                cucumber '**/build/reports/cucumber/*.json'
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

        stage('Notifications') {
            steps {
                sh './gradlew sendSlackNotification sendEmailNotification'
            }
        }
    }

    post {
        failure {
            sh './gradlew sendSlackNotification sendEmailNotification'
        }
    }
}
