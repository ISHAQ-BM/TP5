pipeline {
    agent any

    tools {
        jdk 'java11'
    }

    stages {

        stage('Debug Java') {
            steps {
                bat 'java -version && echo JAVA_HOME=%JAVA_HOME%'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test') {
            steps {
                bat 'gradlew.bat test cucumber'
            }
        }

        stage('Code Analysis') {
            environment {
                JAVA_HOME = tool 'java11'
                PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonarqube'
                }
            }
        }

        stage('Build') {
            steps {
                bat 'gradlew.bat build'
            }
        }
    }
}
