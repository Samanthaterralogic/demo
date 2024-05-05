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

        //stage("Build Application") {
            //steps {
             //   sh "mvn clean package"
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
                        sh "mvn sonar:sonar"
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

        stage("Trivy Scan") {
           steps {
               script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image samanthav1/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }
    }
	
    post {
        failure {
            // Send email notification on failure
            emailext(
                to: 'samveena2023@gmail.com',
                subject: 'Jenkins Pipeline Failed',
                body: "Jenkins pipeline ${env.JOB_NAME} failed. Please check console output for details.",
                attachLog: true
            )
        }
    }
}
