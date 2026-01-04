pipeline {
    agent any

    tools {
        jdk 'java11-latest'
    }
    environment {
        JAVA_HOME = tool 'java11'
        PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
    }

    stages {
        stage('Debug Java') {
            steps {
                bat '''
                java -version
                echo JAVA_HOME=%JAVA_HOME%
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                bat 'gradlew.bat test'  // Only 'test', no separate cucumber task
                junit '**/build/test-results/test/*.xml'
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Replace with your SonarQube server name
                    bat 'gradlew.bat sonarqube'
                }
            }
        }

        stage('Build') {
            steps {
                bat 'gradlew.bat build'
            }
        }

        stage('Deploy') {
            steps {
                bat 'gradlew.bat publish'
            }
        }
    }
}
