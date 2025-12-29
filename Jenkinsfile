pipeline {
    agent any // Or use a specific OpenShift pod template
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Code Quality (PMD)') {
            steps {
                // Runs PMD and generates the XML/HTML report
                sh './mvnw pmd:pmd pmd:check'
            }
            post {
                always {
                    // Publishes the HTML report to the Jenkins UI
                    pmd canRunOnFailed: true, pattern: 'target/pmd.xml'
                    publishHTML(target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'target',
                        reportFiles: 'pmd.html',
                        reportName: 'PMD Report'
                    ])
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh './mvnw test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Deploy to OpenShift') {
            // Only run this if the code is merged into the main branch
            when {
                branch 'main'
            }
            steps {
                script {
                    // Use OpenShift CLI or Jenkins plugin to trigger build
                    sh "oc start-build springboot-camel-app --from-dir=. --follow"
                }
            }
        }
    }
}
