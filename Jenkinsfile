pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: maven
      image: maven:3.9.6-eclipse-temurin-17
      command:
        - cat
      tty: true
"""
        }
    }
    
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
                  //  recordIssues(
                    //            tools: [pmd(pattern: 'target/pmd.xml')],
                      //          enabledForFailure: true
                        // )

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
                        junit testResults: '**/target/surefire-reports/*.xml',
                        allowEmptyResults: true
                    }
                }
        }

        stage('Build JAR') {
            when { branch 'main' }
            steps {
                sh './mvnw clean package -DskipTests'
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
                    sh "oc start-build springboot-camel-app --from-dir=./target --follow"

                    // Optional: Rollout the new deployment
                    sh "oc rollout status deployment/springboot-camel-app"
                }
            }
        }
    }
}
