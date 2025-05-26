pipeline {  
    agent any  
    stages {  
        stage('Test DVWA') {  
            steps {  
                sh 'echo "¡DVWA está lista para análisis!"'  
            }  
        }  
    }  
}  
stage('Analizar con SonarQube') {  
    steps {  
        sh 'sonar-scanner -Dsonar.projectKey=DVWA'  
    }  
    }
    post {  
        always {  
            echo 'Análisis completado.'  
        }  
    }
    tools {  
        sonarQube 'SonarQube'  
    }
    environment {  
        SONAR_HOST_URL = 'http://localhost:9000'  
        SONAR_LOGIN = credentials('admin')
        SONAR_PASSWORD = credentials('admin')
    } 