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

        stage('DAST - OWASP ZAP') {
            steps {
                sh '''
                    echo "=== DAST : OWASP ZAP ==="
                    rm -f zap-report.html zap-report.json

                    # Lancer ZAP en mode quick scan
                    env -u DISPLAY -u XAUTHORITY zaproxy -cmd \
                        -quickurl http://127.0.0.1:3000 \
                        -quickout "$WORKSPACE/zap-report.html" \
                        -quickprogress

                    echo ""
                    echo "=== Rapport ZAP généré ==="
                    ls -lh "$WORKSPACE/zap-report.html"
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'npm-audit-report.json,zap-report.html', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo '✓ Pipeline de sécurité terminé.'
        }
        success {
            echo '✓ Build SUCCESS - Tous les scans ont réussi'
        }
        failure {
            echo '✗ Build FAILED'
        }
    }
}
