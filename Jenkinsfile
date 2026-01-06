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
    success {
                echo "Pipeline réussi ! Envoi de la notification de succès..."
                mail to: 'isaacbelhadjmehdi@gmail.com',
                     subject: "SUCCESS: Pipeline ${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]",
                     body: """Félicitations !
                             Le déploiement de l'API TP5 a été effectué avec succès sur MyMavenRepo.
                             Détails du build : ${env.BUILD_URL}
                             Statut : SUCCESS"""
            }
            failure {
                echo "Pipeline échoué. Envoi de l'alerte..."
                mail to: 'isaacbelhadjmehdi@gmail.com',
                     subject: "FAILURE: Pipeline ${env.JOB_NAME} [Build #${env.BUILD_NUMBER}]",
                     body: """Attention !
                             Le pipeline a échoué. Veuillez vérifier les logs Jenkins pour corriger l'erreur.
                             Lien vers le build : ${env.BUILD_URL}
                             Statut : FAILED"""
            }
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