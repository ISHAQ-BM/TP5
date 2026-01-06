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
                    junit testResults: '**/build/test-results/**/*.xml', allowEmptyResults: true

                    // ✅ AJOUTEZ CECI pour voir la couverture de code
                    jacoco(
                        execPattern: '**/build/jacoco/*.exec',
                        classPattern: '**/build/classes',
                        sourcePattern: '**/src/main/java',
                        exclusionPattern: '**/*Test*.class'
                    )

                    archiveArtifacts artifacts: 'build/reports/**', allowEmptyArchive: true
                }
            }
        }

        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonar'
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
        always {
            junit testResults: '**/build/test-results/**/*.xml', allowEmptyResults: true

            // ✅ MAINTENANT vous pouvez utiliser jacoco()
            jacoco(
                execPattern: '**/build/jacoco/*.exec',
                classPattern: '**/build/classes',
                sourcePattern: '**/src/main/java',
                exclusionPattern: '**/*Test*.class'
            )

            publishHTML(target: [
                allowMissing: false,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'build/reports/cucumber/cucumber-html-reports',
                reportFiles: 'overview-features.html',
                reportName: 'Cucumber Report'
            ])

            archiveArtifacts artifacts: 'build/reports/**', allowEmptyArchive: true
        }
        
    }
}