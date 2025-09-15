pipeline {
  agent {
        docker {
            image 'maven:3.8.5-openjdk-17'
        }
    }
   environment {
        DOCKER_REGISTRY = "abhishek7483/bookmyshow"
        DOCKER_CREDENTIAL_ID = 'abhishek7483'    
    }
  tools
{ maven ('mvn')
     }
  
  stages {
    stage('Checkout') {
      steps {
        git branch: 'master',
          url: 'https://github.com/ABHISHEKJABHI/Book-My-Show.git'
      }
    }
    stage('Build and Test') {
      steps {
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://localhost:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd /opt/maven3/Book-My-Show && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "abhishek7483/bookmyshow:${BUILD_NUMBER}"
      }
      steps {
        script {
          docker.build("${DOCKER_IMAGE}", "/opt/maven3/Book-My-Show")
          docker.withRegistry('https://index.docker.io/v1/', 'abhishek7483') {
            docker.image("${DOCKER_IMAGE}").push()
          }
        }
      }
    }
    stage('Update Deployment File') {
      environment {
        GIT_REPO_NAME = "Book-My-Show"  // CORRECTED: Same repository
        GIT_USER_NAME = "ABHISHEKJABHI"
      }
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          sh '''
            # Configure git
            git config user.email "ashek7944@gmail.com"
            git config user.name "ABHISHEKJABHI"
            
            # Copy deployment file INTO the repository (FIXED)
            cp /opt/dockerimage1/deployment.yml ./deployment.yml
            
            # Update deployment file
            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" ./deployment.yml
            
            # Commit and push changes (FIXED - file is now in repo)
            git add ./deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}"
            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }
  }
}
