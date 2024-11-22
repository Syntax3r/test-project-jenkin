pipeline {
    agent any

    environment {
        GITLEAKS_REPORT = 'gitleaks-report.json'
        TRUFFLEHOG_REPORT = 'trufflehog-report.json'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Run git-secrets scan') {
            steps {
                script {
                    // Initialize git-secrets and register AWS patterns
                    sh 'git secrets --install || true'
                    sh 'git secrets --register-aws || true'

                    // Run git-secrets scan
                    sh 'git secrets --scan --recursive || echo "Potential secrets found by git-secrets."'
                }
            }
        }

        stage('Run Gitleaks scan') {
            steps {
                script {
                    // Run Gitleaks scan and save results in JSON format
                    sh "gitleaks detect --source . --report-format json --report-path ${GITLEAKS_REPORT} || echo 'Potential secrets found by Gitleaks.'"
                }
            }
        }

        stage('Run TruffleHog scan') {
            steps {
                script {
                    // Run TruffleHog scan and output JSON results
                    sh "trufflehog filesystem . --json > ${TRUFFLEHOG_REPORT} || echo 'Potential secrets found by TruffleHog.'"
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: "${GITLEAKS_REPORT}, ${TRUFFLEHOG_REPORT}", allowEmptyArchive: true
            echo 'Secrets scan completed. Reports have been archived.'
        }
        failure {
            echo 'Secret scanning detected potential issues!'
        }
    }
}
