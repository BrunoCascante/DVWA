pipeline {
    agent any
    
    tools {
        sonarQubeScanner 'SonarQube' // Make sure this tool is configured in Jenkins
    }
    
    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN =' squ_c355f075bb75340b8aa9e9d65165f3be41ae5f94' // Recommended to use token instead of login/password
    }
    
    stages {
        stage('Test DVWA') {
            steps {
                sh 'echo "¡DVWA está lista para análisis!"'
            }
        }
        
        stage('Analizar con SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { // Requires SonarQube plugin
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=DVWA \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=${SONAR_AUTH_TOKEN} \
                    '''
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