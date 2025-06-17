pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {

        stage('Verificar herramientas') {
            steps {
                sh 'echo "🔍 Verificando herramientas instaladas..."'
                sh 'which gitleaks && gitleaks version'
                sh 'which dependency-check.sh || echo "⚠️ Dependency-check no está instalado"'
                sh 'which trivy || echo "⚠️ Trivy no está instalado"'
                sh 'which mvn || echo "⚠️ Maven no está instalado"'
                sh '[ -f ./run-zap.sh ] && echo "✅ run-zap.sh encontrado" || echo "❌ run-zap.sh no está en el workspace"'
            }
        }

        stage('Checkout') {
            steps {
                git credentialsId: 'Github-token', url: 'https://github.com/BrunoCascante/DVWA.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "Construcción simple o test build"'
            }
        }

        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    withCredentials([string(credentialsId: 'sonarqube_token', variable: 'SONAR_TOKEN')]) {
                        sh 'mvn clean verify sonar:sonar -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }

        stage('Scan Secrets') {
            steps {
                sh 'gitleaks detect --source=. --report-path=gitleaks-report.json'
            }
        }

        stage('Scan Dependencies') {
            steps {
                sh '''
                    mkdir -p reports
                    mkdir -p /var/lib/jenkins/dependency-check-data
                    dependency-check.sh \
                      --project DVWA \
                      --format HTML \
                      --out reports \
                      --scan . \
                      --data /var/lib/jenkins/dependency-check-data \
                      --log reports/dependency-check.log \
                      --nvdApiKey 20833060-833f-4bb7-8c33-4751cafe4722 \
                      --nvdApiDelay 3000
                '''
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh 'trivy image dvwa'
            }
        }

        stage('Dynamic Analysis (ZAP)') {
            steps {
                sh './run-zap.sh'
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def response = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://localhost:8080', returnStdout: true).trim()
                    if (response != '200') {
                        error("La aplicación no respondió correctamente. Código recibido: ${response}")
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completado exitosamente.'
        }

        failure {
            echo '❌ El pipeline ha fallado. Revisa las etapas anteriores.'
        }

        always {
            echo '📦 Archivos generados en el workspace:'
            sh 'find . -type f'

            archiveArtifacts artifacts: 'reports/**/*.html, reports/**/*.log, **/*.json', allowEmptyArchive: true

            echo '📁 Archivos archivados para revisión.'

            publishHTML([
                reportDir: 'reports',
                reportFiles: 'dependency-check-report.html',
                reportName: 'Dependency Check Report',
                keepAll: true,
                alwaysLinkToLastBuild: true,
                allowMissing: true
            ])
        }
    }
}
