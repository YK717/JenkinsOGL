pipeline {
    agent any

    environment {
        SLACK_WEBHOOK = 'https://hooks.slack.com/services/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'
    }

    // Define the notification function at pipeline level
    def notifySlack(String message, String color = '#000000') {
        script {
            def payload = [
                attachments: [[
                    color: color,
                    text: message,
                    footer: "Job: ${env.JOB_NAME} | Build: ${env.BUILD_NUMBER}"
                ]]
            ]

            httpRequest(
                url: SLACK_WEBHOOK,
                httpMode: 'POST',
                contentType: 'APPLICATION_JSON',
                requestBody: groovy.json.JsonOutput.toJson(payload)
            )
        }
    }

    stages {
        stage('Test') {
            steps {
                script {
                    notifySlack("Starting tests...", "#FFFF00")  // Yellow for started
                    bat 'gradlew.bat test'
                }
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                    cucumber '**/reports/*.json'
                }
                success {
                    notifySlack("Tests passed successfully!", "#36A64F")  // Green for success
                }
                failure {
                    notifySlack("Tests failed!", "#FF0000")  // Red for failure
                }
            }
        }

        stage('SonarQube') {
            steps {
                script {
                    notifySlack("Starting SonarQube analysis...", "#FFFF00")
                    withSonarQubeEnv('sonar') {
                        bat 'gradlew.bat sonar'
                    }
                }
            }
        }

        stage('Code Quality') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        notifySlack("Quality Gate failed: ${qualityGate.status}", "#FF0000")
                        error "Pipeline failed due to Quality Gate failure: ${qualityGate.status}"
                    } else {
                        notifySlack("Quality Gate passed successfully!", "#36A64F")
                    }
                }
            }
        }

        stage('Build') {
            steps {
                script {
                    notifySlack("Starting build process...", "#FFFF00")
                    bat 'gradlew.bat build'
                    bat 'gradlew.bat javadoc'
                    archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
                    archiveArtifacts artifacts: 'build/docs/**', allowEmptyArchive: true
                }
            }
            post {
                success {
                    notifySlack("Build completed successfully!", "#36A64F")
                }
                failure {
                    notifySlack("Build failed!", "#FF0000")
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    notifySlack("Starting deployment...", "#FFFF00")
                    bat 'gradlew.bat publish'
                }
            }
            post {
                success {
                    notifySlack("Deployment completed successfully!", "#36A64F")
                }
                failure {
                    notifySlack("Deployment failed!", "#FF0000")
                }
            }
        }
    }

    post {
        success {
            notifySlack("Pipeline completed successfully! 🎉", "#36A64F")
        }
        failure {
            notifySlack("Pipeline failed! 🚨", "#FF0000")
        }
        always {
            notifySlack("Pipeline finished. Duration: ${currentBuild.durationString}")
        }
    }
}