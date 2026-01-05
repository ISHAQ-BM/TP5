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
                // 1. Lancement des tests unitaires
                bat 'gradlew.bat clean test jacocoTestReport --rerun-tasks'
            }
            post {
                always {
                    // 2. Archivage des résultats et 3. Rapports Cucumber
                    junit testResults: 'build/test-results/test/*.xml', allowEmptyResults: true
                    archiveArtifacts artifacts: 'build/reports/**', allowEmptyArchive: true
                }
            }
        }

        stage('Code Analysis') {
            steps {
                // Analyse SonarQube
                withSonarQubeEnv('SonarQube') {
                    bat 'gradlew.bat sonar'
                }
            }
        }

        stage('Code Quality') {
            steps {
                // Vérification de l'état de Quality Gates
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build') {
            steps {
                // 1. Génération Jar et 2. Génération Documentation
                bat 'gradlew.bat jar javadoc'
            }
            post {
                success {
                    // 3. Archivage du fichier Jar et de la documentation
                    archiveArtifacts artifacts: 'build/libs/*.jar, build/docs/javadoc/**', fingerprint: true
                }
            }
        }

        stage('Deploy') {
            steps {
                // Déploiement réussi vers MyMavenRepo
                bat 'gradlew.bat publish'
            }
        }
    }

    post {
        // Notifications par mail
        success {
            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "SUCCESS - ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "Le déploiement a été effectué avec succès."
        }
        failure {
            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "FAILURE - ${env.JOB_NAME} Build #${env.BUILD_NUMBER}",
                 body: "Le pipeline a échoué. Veuillez vérifier les logs."
        }
    }
}