pipeline {
    agent any
    
    tools {
        maven 'Maven 3.9.9' // Asegúrate que coincide exactamente con el nombre en Jenkins
    }
    
    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = 'squ_c355f075bb75340b8aa9e9d65165f3be41ae5f94'// Credencial almacenada en Jenkins
    }
    
    stages {
        stage('SonarQube Analysis') {
            steps {
                script {
                    // Ejecuta Maven con los parámetros explícitos
                    sh """
                        mvn clean verify sonar:sonar \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN} \
                        -Dsonar.projectKey=DVWA \
                        -Dsonar.projectName=DVWA
                    """
                }
            }
        }
    }
}