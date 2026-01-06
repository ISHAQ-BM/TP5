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
            echo "Pipeline finished with status: SUCCESS"
            echo "SonarQube Report: http://localhost:9000/dashboard?id=TP5"

            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "✅ SUCCESS: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                 body: """Build réussi!

Projet: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Quality Gate: ${env.QUALITY_GATE_STATUS ?: 'N/A'}
Durée: ${currentBuild.durationString}

Voir les détails: ${env.BUILD_URL}
SonarQube: http://localhost:9000/dashboard?id=TP5

---
Jenkins Automation"""
        }
        failure {
            echo "Pipeline finished with status: FAILURE"

            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "❌ FAILURE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                 body: """Build échoué!

Projet: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Durée: ${currentBuild.durationString}

Voir les logs: ${env.BUILD_URL}console

---
Jenkins Automation"""
        }
        unstable {
            echo "Pipeline finished with status: UNSTABLE"

            mail to: 'isaacbelhadjmehdi@gmail.com',
                 subject: "⚠️ UNSTABLE: ${env.JOB_NAME} - Build #${env.BUILD_NUMBER}",
                 body: """Build instable!

Projet: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Durée: ${currentBuild.durationString}

Certains tests ont échoué ou le Quality Gate n'est pas passé.

Voir les détails: ${env.BUILD_URL}

---
Jenkins Automation"""
        }
    }
}
```

---

## Configuration SMTP dans Jenkins (obligatoire):

### **1. Configurez le serveur email:**

**Manage Jenkins** → **Configure System** → Cherchez **"E-mail Notification"**

#### **Pour Gmail:**
```
SMTP server: smtp.gmail.com
```

Cliquez sur **Advanced**:
```
☑ Use SMTP Authentication
User Name: votre-email@gmail.com
Password: [votre mot de passe d'application]
☑ Use SSL
SMTP Port: 465
```

**OU** avec TLS:
```
☑ Use SMTP Authentication
User Name: votre-email@gmail.com
Password: [votre mot de passe d'application]
☑ Use TLS
SMTP Port: 587
```

#### **Mot de passe d'application Gmail:**
1. Allez sur: https://myaccount.google.com/apppasswords
2. Créez un nouveau mot de passe d'application
3. Utilisez ce mot de passe (pas votre mot de passe Gmail normal)

---

### **2. Testez la configuration:**

Dans la même page, en bas:

**Test configuration by sending test e-mail**
- Entrez: `isaacbelhadjmehdi@gmail.com`
- Cliquez sur **"Test configuration"**

Vous devriez recevoir un email de test.

---

## Autres providers SMTP:

**Outlook/Hotmail:**
```
SMTP: smtp-mail.outlook.com
Port: 587 (TLS)
```

**Yahoo:**
```
SMTP: smtp.mail.yahoo.com
Port: 587 (TLS)