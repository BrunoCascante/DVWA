pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
    }

    stages {

        stage('Verificar herramientas') {
            steps {
                sh '''
                    echo "🔍 Verificando herramientas instaladas..."
                    which gitleaks && gitleaks version || echo "⚠️ Gitleaks no está instalado"
                    which dependency-check.sh || echo "⚠️ Dependency-check no está instalado"
                    which trivy || echo "⚠️ Trivy no está instalado"
                    which mvn || echo "⚠️ Maven no está instalado"
                    [ -f ./run-zap.sh ] && echo "✅ run-zap.sh encontrado" || echo "❌ run-zap.sh no está en el workspace"
                '''
            }
        }

        stage('Checkout') {
            steps {
                git credentialsId: 'Github-token', url: 'https://github.com/BrunoCascante/DVWA.git'
            }
        }

        stage('Build') {
            steps {
                sh 'echo "🏗️ Construcción simple o test build"'
            }
        }

        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    withCredentials([string(credentialsId: 'sonarqube_token', variable: 'SONAR_TOKEN')]) {
                        sh 'mvn clean verify sonar:sonar -Dsonar.login=$SONAR_TOKEN || echo "⚠️ SonarQube falló, revisa los logs."'
                    }
                }
            }
        }

        stage('Scan Secrets') {
            steps {
                sh 'gitleaks detect --source=. --report-path=gitleaks-report.json || echo "⚠️ Gitleaks encontró secretos o falló."'
            }
        }

        stage('Scan Dependencies') {
            steps {
                sh '''
                    mkdir -p reports
                    dependency-check.sh \
                      --project DVWA \
                      --format HTML \
                      --out reports \
                      --scan . \
                      --log reports/dependency-check.log \
                      --data /var/lib/jenkins/dependency-check-data \
                      --nvdApiKey 20833060-833f-4bb7-8c33-4751cafe4722 \
                      --nvdApiDelay 3000 || echo "⚠️ Dependency-Check falló, revisa los logs."
                '''
            }
        }

        stage('Scan Docker Image') {
            steps {
                sh '''
                    trivy image -f json -o reports/trivy-report.json ghcr.io/digininja/dvwa || echo "⚠️ Trivy terminó con errores o vulnerabilidades."
                '''
            }
        }


        stage('Dynamic Analysis (ZAP)') {
            steps {
                sh './run-zap.sh || echo "⚠️ ZAP script falló, revisar logs."'
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def response = sh(script: 'curl -s -o /dev/null -w "%{http_code}" http://localhost:8080', returnStdout: true).trim()
                    if (response != '200') {
                        echo "⚠️ La aplicación no respondió correctamente. Código recibido: ${response}"
                    } else {
                        echo "✅ Aplicación activa. Código HTTP: ${response}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo '📦 Archivos generados en el workspace:'
            sh 'find . -type f'

            archiveArtifacts artifacts: 'reports/**/*.html, reports/**/*.log, **/*.json', allowEmptyArchive: true

            echo '📁 Archivos archivados para revisión.'

            script {
                if (fileExists('reports/dependency-check-report.html')) {
                    publishHTML([
                        reportDir: 'reports',
                        reportFiles: 'dependency-check-report.html',
                        reportName: 'Dependency Check Report',
                        keepAll: true,
                        alwaysLinkToLastBuild: true,
                        allowMissing: true
                    ])
                } else {
                    echo "⚠️ No se encontró el archivo dependency-check-report.html para publicar"
                }
            }
        }

        success {
            echo '✅ Pipeline completado exitosamente.'
        }

        failure {
            echo '❌ El pipeline ha fallado en alguna etapa. Revisa los resultados.'
        }
    }
}
