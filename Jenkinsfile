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
                        // استخدام --rerun-tasks لضمان توليد ملفات الـ XML في كل مرة
                        bat 'gradlew.bat clean test jacocoTestReport --rerun-tasks'
                    }
                    post {
                        always {
                            junit 'build/test-results/test/*.xml'
                        }
                    }
                }

                stage('Code Analysis') {
                    steps {
                        // تأكد أن اسم 'SonarQube' يطابق الاسم المعرف في Jenkins -> Manage Jenkins -> System
                        withSonarQubeEnv('SonarQube') {
                            bat 'gradlew.bat sonar'
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
