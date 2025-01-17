pipeline {
    agent any
    tools {
        maven 'M3'
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credential' // Replace with your Jenkins Docker credentials ID
        DOCKER_IMAGE_NAME = 'amirulafiqj/jenkins-demo' // Replace with your Docker Hub username and repo name
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-credentials' // Replace with your Jenkins Kubernetes credentials ID this one later
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
                bat 'docker build -t %DOCKER_IMAGE_NAME%:%BUILD_NUMBER% .'
                withDockerRegistry(credentialsId: "${DOCKER_CREDENTIALS_ID}", url: '') {
                    bat 'docker push %DOCKER_IMAGE_NAME%:%BUILD_NUMBER%'
                        }
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
                                kubectl set image deployment/jenkins-demo jenkins-demo=${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} --record
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


