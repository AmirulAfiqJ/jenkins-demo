pipeline {
    agent any
    tools {
        maven 'M3'
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credential' // Replace with your Jenkins Docker credentials ID
        DOCKER_IMAGE_NAME = 'amirulafiqj/jenkins-demo' // Replace with your Docker Hub username and repo name
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-credential' // Replace with your Jenkins Kubernetes credentials ID this one later
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                dir('jenkins-demo') {
                    bat 'mvn clean install'
                }
            }
        }

        stage('Test') {
            steps {
                dir('jenkins-demo') {
                    bat 'mvn test'
                }
            }
        }

        stage('Code Analysis') {
            steps {
                dir('jenkins-demo') {
                    withSonarQubeEnv('SonarQube') {
                        bat 'mvn sonar:sonar'
                    }
                }
            }
        }

        stage('Docker Build and Push') {
            steps {
                script {
                    dir('jenkins-demo') {
                        bat 'docker build -t amirulafiqj/jenkins-demo:%BUILD_NUMBER% .'       
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                script {
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIALS_ID}"]) {
                        dir('jenkins-demo') {
                            bat """
				kubectl apply -f deployment.yaml
                                kubectl rollout status deployment/jenkins-demo
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}


