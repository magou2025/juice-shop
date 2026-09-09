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

        stage('DAST - OWASP ZAP') {
            steps {
                sh '''
                    echo "===== DAST : OWASP ZAP ====="

                    rm -f "$WORKSPACE/zap-report.html"

                    env -u DISPLAY -u XAUTHORITY zaproxy -cmd \
                      -quickurl http://127.0.0.1:3000 \
                      -quickout "$WORKSPACE/zap-report.html" \
                      -quickprogress

                    echo "===== Rapport ZAP généré ====="
                    ls -lh "$WORKSPACE/zap-report.html"
                '''
            }
        }

        stage('Archive Security Reports') {
            steps {
                archiveArtifacts artifacts: 'npm-audit-report.json,zap-report.html',
                                 allowEmptyArchive: false
            }
        }
    }

    post {
        always {
            echo 'Pipeline de sécurité terminé.'
        }
    }
}
