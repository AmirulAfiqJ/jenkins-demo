pipeline {
    agent any
    tools {
        maven 'M3'
    }

    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credential' // Replace with your Jenkins Docker credentials ID
        DOCKER_IMAGE_NAME = 'amirulafiqj/jenkins-demo' // Replace with your Docker Hub username and repo name
        KUBECONFIG_CREDENTIALS_ID = 'kubeconfig-credential' // Replace with your Jenkins Kubernetes credentials ID
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
                        // Build Docker image with both build number and latest tags
                        bat """
                            docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} -t ${DOCKER_IMAGE_NAME}:latest .
                        """
                        // Log in to Docker Hub and push both tags
                        withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS_ID}"]) {
                            bat """
                                docker push ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}
                                docker push ${DOCKER_IMAGE_NAME}:latest
                            """
                        }
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                script {
                    // Use Kubernetes credentials to apply deployment and set the image dynamically
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIALS_ID}"]) {
                        dir('jenkins-demo') {
                            bat """
                                kubectl apply -f deployment.yaml
                                kubectl set image deployment/jenkins-demo jenkins-container=${DOCKER_IMAGE_NAME}:latest --record
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


