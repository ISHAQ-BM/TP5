pipeline {
    agent any
    tools { jdk 'java11' }
    environment {
        JAVA_HOME = tool 'java11'
        PATH = "${env.JAVA_HOME}\\bin;${env.PATH}"
        EMAIL_RECIPIENTS = 'isaacbelhadjmehdi@gmail.com'
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
        stage('Code Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonar'
                }
            }
        }
        stage('Code Quality') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()
                        env.QUALITY_GATE_STATUS = qg.status
                        if (qg.status != 'OK') {
                            echo "Quality Gate failed: ${qg.status}"
                            // Ne pas bloquer, mais noter l'échec
                        }
                    }
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
            script {
                def qualityGateMsg = env.QUALITY_GATE_STATUS == 'OK' ? '✅ PASSED' : '⚠️ FAILED'

                emailext(
                    subject: "✅ SUCCESS: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                    body: """
                        <h2>✅ Build Successful!</h2>
                        <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                        <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                        <p><strong>Quality Gate:</strong> ${qualityGateMsg}</p>

                        <h3>📊 Reports:</h3>
                        <ul>
                            <li><a href="${env.BUILD_URL}testReport">Test Results</a></li>
                            <li><a href="${env.BUILD_URL}Cucumber_Report">Cucumber Report</a></li>
                            <li><a href="http://localhost:9000/dashboard?id=TP5">SonarQube Analysis</a></li>
                            <li><a href="${env.BUILD_URL}artifact/build/reports/jacoco/test/html/index.html">Code Coverage</a></li>
                        </ul>

                        <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                        <p><a href="${env.BUILD_URL}">View Full Build</a></p>
                    """,
                    to: "${EMAIL_RECIPIENTS}",
                    mimeType: 'text/html'
                )
            }
        }
        failure {
            emailext(
                subject: "❌ FAILURE: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>❌ Build Failed!</h2>
                    <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> FAILED</p>

                    <h3>📋 Details:</h3>
                    <p>The pipeline failed during execution. Please check the console output for details.</p>

                    <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                    <p><a href="${env.BUILD_URL}console">View Console Output</a></p>
                    <p><a href="${env.BUILD_URL}">View Full Build</a></p>
                """,
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
        unstable {
            emailext(
                subject: "⚠️ UNSTABLE: Pipeline ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <h2>⚠️ Build Unstable!</h2>
                    <p><strong>Project:</strong> ${env.JOB_NAME}</p>
                    <p><strong>Build Number:</strong> ${env.BUILD_NUMBER}</p>
                    <p><strong>Status:</strong> UNSTABLE</p>

                    <h3>📋 Details:</h3>
                    <p>The build completed but some tests failed or quality gates were not met.</p>

                    <h3>📊 Reports:</h3>
                    <ul>
                        <li><a href="${env.BUILD_URL}testReport">Test Results</a></li>
                        <li><a href="http://localhost:9000/dashboard?id=TP5">SonarQube Analysis</a></li>
                    </ul>

                    <p><strong>Duration:</strong> ${currentBuild.durationString}</p>
                    <p><a href="${env.BUILD_URL}">View Full Build</a></p>
                """,
                to: "${EMAIL_RECIPIENTS}",
                mimeType: 'text/html'
            )
        }
        always {
            echo "Pipeline finished with status: ${currentBuild.result}"
            echo "SonarQube Report: http://localhost:9000/dashboard?id=TP5"
        }
    }
}