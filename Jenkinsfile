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
                    cucumber 'build/reports/cucumber/*.json'
                }
            }
        }

        stage('SonarQube') {
            steps {
                // Use the SonarQube environment wrapper
                withSonarQubeEnv('sonar') { // Replace 'sonar' with the name of your configured SonarQube server in Jenkins
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
}

// test
// test2