pipeline {
    agent any 
    tools {
        maven "M3"
    }

    environment {
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_AUTH_TOKEN = credentials('sonar-token-id') 
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/AmirulAfiqJ/jenkins-demo.git' 
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install' 
            }
        }
        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
        }
        stage('SonarQube Analysis') { 
            steps {
                script {
                    withSonarQubeEnv('SonarQube') { 
                        sh """
                        mvn sonar:sonar \
                             -Dsonar.projectKey=test-demo \
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
            echo 'Pipeline execution complete.'
        }
        success {
            echo 'Build, Test, and Static Code Analysis stages succeeded!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
