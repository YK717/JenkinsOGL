pipeline {
    agent any

    environment {
        NOTIFICATION_EMAIL = 'la_bengherbia@esi.dz'
        SLACK_CHANNEL = '#tp-jenkins'
        TEST_RESULTS_PATH = 'build/test-results/test/*.xml'
        CUCUMBER_REPORTS_PATH = '**/reports/*.json'
        JAR_ARTIFACTS_PATH = '**/*.jar'
        DOCS_ARTIFACTS_PATH = 'build/docs/**'
        SUCCESS_SUBJECT = "Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        FAILURE_SUBJECT = "Pipeline Failure: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        SUCCESS_BODY = """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} completed successfully.
                         \nCheck the details at: ${env.BUILD_URL}"""
        FAILURE_BODY = """The pipeline for ${env.JOB_NAME} build #${env.BUILD_NUMBER} failed.
                         \nCheck the details at: ${env.BUILD_URL}"""
        QUALITY_GATE_ERROR = "Pipeline failed due to Quality Gate failure: %s"
    }

    stages {
        stage('Test') {
            steps {
                script {
                    bat 'gradlew.bat test'
                }
            }
            post {
                always {
                    junit env.TEST_RESULTS_PATH
                    cucumber env.CUCUMBER_REPORTS_PATH
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
                     def qualityGate = waitForQualityGate()
                     if (qualityGate.status != 'OK') {
                         error String.format(env.QUALITY_GATE_ERROR, qualityGate.status)
                     }
                 }
             }
        }

        stage('Build') {
             steps {
                 script {
                     bat 'gradlew.bat build'
                     bat 'gradlew.bat javadoc'
                     archiveArtifacts artifacts: env.JAR_ARTIFACTS_PATH, allowEmptyArchive: true
                     archiveArtifacts artifacts: env.DOCS_ARTIFACTS_PATH, allowEmptyArchive: true
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

    post {
        success {
            mail to: env.NOTIFICATION_EMAIL,
                 subject: env.SUCCESS_SUBJECT,
                 body: env.SUCCESS_BODY

            slackSend channel: env.SLACK_CHANNEL,
                      color: 'good',
                      message: "${env.SUCCESS_SUBJECT}\nDetails: ${env.BUILD_URL}"
        }
        failure {
            mail to: env.NOTIFICATION_EMAIL,
                 subject: env.FAILURE_SUBJECT,
                 body: env.FAILURE_BODY

            slackSend channel: env.SLACK_CHANNEL,
                      color: 'danger',
                      message: "${env.FAILURE_SUBJECT}\nDetails: ${env.BUILD_URL}"
        }
    }
}