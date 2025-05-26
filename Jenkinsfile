pipeline {
    agent any

    tools {
        maven 'Maven 3.9.6'
         sonarRunner 'SonarQubeScanner'
    }

    stages {
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }
    }
}
