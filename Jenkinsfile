pipeline {
    agent any

    tools {
        maven 'Maven 3.9.9'  // Asegúrate que este nombre coincide con tu instalación en Jenkins
    }

    environment {
        // Configuración recomendada para SonarQube
        SONAR_HOST_URL = 'http://localhost:9000'  // Cambia si tu SonarQube está en otra URL
        SONAR_AUTH_TOKEN = credentials('sonar-token')  // Credencial guardada en Jenkins
    }

    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {  // Nombre de la configuración en Jenkins
                    sh 'mvn clean verify sonar:sonar -Dsonar.host.url=$SONAR_HOST_URL -Dsonar.login=$SONAR_AUTH_TOKEN'
                }
            }
        }
    }
}