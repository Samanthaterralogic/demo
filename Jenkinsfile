pipeline {
    agent any
    
    tools {
        jdk 'java17'
        maven 'Maven3'
    }
    
    environment {
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "samanthav1"
        DOCKER_PASS = 'dockerhub'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        
    }
    
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }

        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/Ashfaque-9x/register-app'
            }
        }

        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }

        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') { 
                        sh 'mvn sonar:sonar -Dsonar.host.url=http://192.168.199.13:9000'
                    }
                }
            }
        }

        stage("Build Docker Image") {
            steps {
                script {
                    // Authenticate with Docker Hub using stored credentials
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        // Build the Docker image
                        def dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage("Push Docker Image") {
            steps {
                script {
                    // Authenticate with Docker Hub using stored credentials
                    withDockerRegistry(credentialsId: 'dockerhub') {
                        // Push the Docker image to the registry
                        def dockerImage = docker.image("${IMAGE_NAME}:${IMAGE_TAG}")
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy To Kubernetes') {
           steps {
               script {
                   kubeconfig(caCertificate: '', clusterName: 'minikube', contextName: 'context_info', credentialsId: 'jenkins-secret', namespace: 'default', restrictKubeConfigAccess: false, serverUrl: 'https://192.168.49.2:8443') {
                       sh "kubectl apply -f /cd/deploy.yaml"
                       sh "kubectl apply -f /cd/cd-service.yaml"
                       
                }
            }
        }    
    }    
}
        

    
    post {
        always {
            // Send email notification
            emailext(
                subject: "Pipeline Status: ${BUILD_NUMBER}",
                body: '''<html>
                            <body>
                                <p>Build Status: ${BUILD_STATUS}</p>
                                <p>Build Number: ${BUILD_NUMBER}</p>
                                <p>Check the <a href= "${BUILD_URL}">console output</a>.</p>
                            </body>
                         </html>''',
                to: 'samveena2023@gmail.com',
                from: 'jenkins@example.com',
                replyTo: 'jenkins@example.com',
                mimeType: 'text/html'
            )
        }
    }
}   

