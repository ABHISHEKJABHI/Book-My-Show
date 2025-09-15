// Jenkins Declarative Pipeline for Git, Maven, SonarQube, and Docker
pipeline {
    agent {
        // Use a Docker agent for the build process to ensure a consistent environment
        docker {
            image 'maven:3.8.5-openjdk-17'
        }
    }

    // Define environment variables, including Docker registry details
    environment {
        // Replace with your Docker Hub username and repository name
        DOCKER_REGISTRY = "abhishek7483/bookmyshow"
        DOCKER_CREDENTIAL_ID = 'abhishek7483'
        SONAR_HOST_URL = 'http://localhost:9000'
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Check out the code from your Git repository
                git branch: 'master',
                  url: 'https://github.com/ABHISHEKJABHI/Book-My-Show.git'
            }
        }

        stage('Build with Maven') {
            steps {
                // Clean the project and compile the source code with Maven
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('sonarserver') {
                   sh """
                   mvn sonar:sonar \
                   -Dsonar.projectKey=scan-code \
                   -Dsonar.projectName='scan-code' \
                   -Dsonar.host.url=${SONAR_HOST_URL}
                   """ 
                }
            }
        }
        
        stage('Wait for Quality Gate') {
            steps {
                // Wait for the SonarQube quality gate to pass
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Application') {
            steps {
                // Package the application after quality gate passes
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build the Docker image, tagging it with the build number
                script {
                    def appImage = docker.build("${env.DOCKER_REGISTRY}:${env.BUILD_NUMBER}", ".")
                    appImage.tag("${env.DOCKER_REGISTRY}:latest")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                // Push the newly built Docker images to the registry
                script {
                    docker.withRegistry("https://index.docker.io/v1/", env.DOCKER_CREDENTIAL_ID) {
                        docker.image("${env.DOCKER_REGISTRY}:${env.BUILD_NUMBER}").push()
                        docker.image("${env.DOCKER_REGISTRY}:latest").push()
                    }
                }
            }
        }
    }
    
    // Post-build actions to clean up local resources
    post {
        always {
            echo "Pipeline finished."
            cleanWs() // Clean workspace after build
        }
        success {
            echo "Pipeline succeeded."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
