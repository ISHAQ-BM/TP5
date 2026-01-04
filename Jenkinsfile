pipeline {
    agent any

    tools {
        jdk 'java11'
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

        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [
                        [$class: 'CheckoutOption', timeout: 30], // زيادة وقت الانتظار لـ 30 دقيقة
                        [$class: 'CloneOption', timeout: 30, shallow: true, noTags: true] // سحب آخر نسخة فقط لتقليل الحجم
                    ],
                    userRemoteConfigs: [[url: 'https://github.com/ISHAQ-BM/TP5', credentialsId: 'github_token']]
                ])
            }
        }

        stage('Test') {
            steps {
                bat 'gradlew.bat test jacocoTestReport'
            }
            post {
                always {
                    // المسار الصحيح لنتائج اختبارات Gradle
                    junit 'build/test-results/test/*.xml'
                }
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
