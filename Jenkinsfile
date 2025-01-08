pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                script {
                    sh './gradlew test'
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
                    sh './gradlew sonar'
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
                     sh './gradlew build'
                     sh './gradlew javadoc'
                     archiveArtifacts artifacts: '**/*.jar', allowEmptyArchive: true
                     archiveArtifacts artifacts: 'build/docs/**', allowEmptyArchive: true
                 }
             }
         }
         stage('Deploy') {
             steps {
                 script {
                     sh './gradlew publish'
                 }
             }
         }
    }
    post {
            success {
                mail to: 'lm_arabet@esi.dz',
                     subject: 'Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                     body: """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully.
                             \nCheck the details at: ${env.BUILD_URL}"""

                slackSend channel: '#jenkins-notifications',
                                      color: 'good',
                                      message: "SUCCESS: ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully. \nDetails: ${env.BUILD_URL}"
            }
            failure {
                mail to: 'lm_arabet@esi.dz',
                     subject: 'Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}',
                     body: """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} failed.
                             \nCheck the details at: ${env.BUILD_URL}"""

                slackSend channel: '#jenkins-notifications',
                                      color: 'danger',
                                      message: "FAILURE: ${env.JOB_NAME} build #${env.BUILD_NUMBER} failed. \nDetails: ${env.BUILD_URL}"

            }
        }
}
