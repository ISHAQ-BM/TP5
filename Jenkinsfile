pipeline {
    agent any
    tools { jdk 'java11' }
    environment {
        JAVA_HOME = tool 'java11'
        PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Test') {
            steps {
                bat 'gradlew.bat clean test jacocoTestReport --rerun-tasks'
            }
            post {
                always {
                    // Use a very broad pattern to ensure Jenkins finds the XMLs
                    junit testResults: '**/build/test-results/**/*.xml', allowEmptyResults: true
                    archiveArtifacts artifacts: 'build/reports/**', allowEmptyArchive: true
                }
            }
        }
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonar -Dsonar.branch.name=main'
                }
            }
        }
        stage('Code Quality') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build') {
            steps {
                bat 'gradlew.bat jar javadoc'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
                }
            }
        }
        stage('Deploy') {
            steps {
                bat 'gradlew.bat publish'
            }
        }
    }
    post {
        success {
            echo "Pipeline finished successfully!"
            // mail to: 'isaacbelhadjmehdi@gmail.com', subject: "SUCCESS", body: "TP5 Deployed"
        }
    }
}