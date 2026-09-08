pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SCA - npm audit') {
            steps {
                sh '''
                    echo "===== SCA : npm audit ====="

                    npm audit --json > npm-audit-report.json || true

                    echo "===== Résumé npm audit ====="
                    npm audit || true
                '''
            }
        }

        stage('Archive SCA report') {
            steps {
                archiveArtifacts artifacts: 'npm-audit-report.json',
                                 allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo 'Pipeline SCA terminé.'
        }
    }
}
