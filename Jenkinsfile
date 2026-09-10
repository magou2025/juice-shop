pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'echo "✓ Code récupéré depuis GitHub"'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    echo "=== Installation des dépendances ==="
                    npm install --silent
                '''
            }
        }

        stage('SCA - npm audit') {
            steps {
                sh '''
                    echo "=== SCA : npm audit ==="
                    npm audit --json > npm-audit-report.json || true
                    echo ""
                    echo "=== Résumé npm audit ==="
                    npm audit || true
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'npm-audit-report.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo '✓ Pipeline SCA terminé.'
        }
        success {
            echo '✓ Build SUCCESS'
        }
        failure {
            echo '✗ Build FAILED'
        }
    }
}
