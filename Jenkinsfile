pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                script {
                    bat 'gradlew.bat test'
                }
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                    cucumber '**/reports/*.json'
                }
            }
        }
        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                    bat 'gradlew.bat sonar'
                }
            }
        }

        stage('Code Quality') {
             steps {
                 script {
                     def qualityGate = waitForQualityGate() // Wait for SonarQube's analysis result
                     if (qualityGate.status != 'OK') {
                         error "Pipeline failed due to Quality Gate failure: ${qualityGate.status}"
                     }
                 }
             }
        }
         stage('Build') {
             steps {
                 script {
                     bat 'gradlew.bat build'
                     bat 'gradlew.bat javadoc'
                     archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
                     archiveArtifacts artifacts: 'build/docs/**', allowEmptyArchive: true
                 }
             }
         }
         stage('Deploy') {
             steps {
                 script {
                     bat 'gradlew.bat publish'
                 }
             }
         }
    }
    /*post {
            success {
                mail to: 'la_bengherbia@esi.dz',
                     subject: "Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully.
                             \nCheck the details at: ${env.BUILD_URL}"""

                slackSend channel: '#jenkins-notifications',
                          color: 'good',
                          message: "SUCCESS: ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully. \nDetails: ${env.BUILD_URL}"
            }
            failure {
                mail to: 'la_bengherbia@esi.dz',
                     subject: "Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                     body: """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} failed.
                             \nCheck the details at: ${env.BUILD_URL}"""

                slackSend channel: '#jenkins-notifications',
                          color: 'danger',
                          message: "FAILURE: ${env.JOB_NAME} build #${env.BUILD_NUMBER} failed. \nDetails: ${env.BUILD_URL}"
            }
        }*/
}