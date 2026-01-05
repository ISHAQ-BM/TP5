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
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/ISHAQ-BM/TP5', credentialsId: 'github_token']]
                ])
            }
        }

        stage('Test') {
            steps {
                // Phase 2.1: Launch tests and generate coverage
                bat 'gradlew.bat clean test jacocoTestReport --rerun-tasks'
            }
            post {
                always {
                    // Phase 2.1: Archive results using a recursive pattern to find XMLs
                    junit testResults: 'build/test-results/**/*.xml', allowEmptyResults: true
                    archiveArtifacts artifacts: 'build/reports/**', allowEmptyArchive: true
                }
            }
        }

        stage('Code Analysis') {
            steps {
                // Phase 2.2: SonarQube analysis
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                // Phase 2.3: Verify Quality Gate status
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                // Phase 2.4: Generate Jar and Javadoc
                bat 'gradlew.bat jar javadoc'
            }
            post {
                success {
                    // Phase 2.4: Archive Jar and Javadoc
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                // Phase 2.5: Deploy to MyMavenRepo
                bat 'gradlew.bat publish'
            }
        }
    }

    post {
        // Phase 2.6: Notifications
        success {
            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "SUCCESS - ${env.JOB_NAME}",
                 body: "Project deployed successfully to MyMavenRepo."
        }
        failure {
            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "FAILURE - ${env.JOB_NAME}",
                 body: "The pipeline failed. Please check the Jenkins logs."
        }
    }
}