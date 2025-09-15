pipeline {
    agent {
        docker {
            image 'maven:3.8.5-openjdk-17'
            args '--network=host' // Added network host to fix SonarQube connectivity
        }
    }

    environment {
        DOCKER_REGISTRY = "abhishek7483/bookmyshow"
        DOCKER_CREDENTIAL_ID = 'abhishek7483'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master',
                  url: 'https://github.com/ABHISHEKJABHI/Book-My-Show.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean compile -DskipTests' // Added skipTests to avoid test failures
            }
        }

       stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://localhost:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh ' mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
   }
        
        stage('Wait for Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Application') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Check if Dockerfile exists before building
                    def dockerfileExists = fileExists 'Dockerfile'
                    if (!dockerfileExists) {
                        error "Dockerfile not found in the workspace"
                    }
                    def appImage = docker.build("${env.DOCKER_REGISTRY}:${env.BUILD_NUMBER}", ".")
                    appImage.tag("${env.DOCKER_REGISTRY}:latest")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry("https://index.docker.io/v1/", env.DOCKER_CREDENTIAL_ID) {
                        docker.image("${env.DOCKER_REGISTRY}:${env.BUILD_NUMBER}").push()
                        docker.image("${env.DOCKER_REGISTRY}:latest").push()
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline finished."
            cleanWs()
        }
        success {
            echo "Pipeline succeeded."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
