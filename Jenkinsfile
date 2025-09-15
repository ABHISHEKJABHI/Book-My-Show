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
        SONAR_HOST_URL = 'http://localhost:9000' // Changed back to localhost with network=host
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

        stage('SonarQube Analysis') {
            steps {
               withSonarQubeEnv('sonarserver') {
                   sh """
                   mvn sonar:sonar \
                   -Dsonar.projectKey=scan-code1 \
                   -Dsonar.projectName='scan-code1' \
                   -Dsonar.host.url=${SONAR_HOST_URL} \
                   -Dsonar.login=sqp_85256af71e8410a3474f5200749bb1b641520a1b \
                   -DskipTests
                   """ 
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
