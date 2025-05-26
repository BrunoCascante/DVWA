pipeline {
    agent any
    
    tools {
        // Nombre DEBE coincidir con la instalación en Jenkins
        sonarScanner 'SonarQube' 
    }
    
    environment {
        // Usar IP/hostname accesible desde Jenkins
        SONAR_HOST_URL = 'http://localhost:9000' 
        // Usar credencial almacenada (mejor práctica)
        SONAR_AUTH_TOKEN = credentials('sonar-token') 
    }
    
    stages {
        stage('Test DVWA') {
            steps {
                sh 'echo "¡DVWA está lista para análisis!"'
            }
        }
        
        stage('Analizar con SonarQube') {
            steps {
                script {
                    // Forma más robusta usando la herramienta instalada
                    def scannerHome = tool 'SonarQube'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=DVWA \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_AUTH_TOKEN}
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo 'Análisis completado.'
        }
    }
}
