pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'SonarQube'
        SONARQUBE_TOKEN = credentials('sonar-token')  // <- este es el ID correcto
    }

    stages {
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
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
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
                sh 'dependency-check.sh --project DVWA --format HTML --out reports/ --scan .'
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
        always {
            script {
                echo "Archivos generados:"
                sh 'find . -type f'
            }
            archiveArtifacts artifacts: 'reports/**/*.html, **/*.json', allowEmptyArchive: true
        }
    }
}
