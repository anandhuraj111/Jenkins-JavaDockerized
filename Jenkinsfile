pipeline {
  agent any

  environment {
    IMAGE = 'anandhuraj111/containerized-java'
    DOCKER_CREDS = credentials('docker-id')
  }

  stages {

    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/bhuvan-raj/Jenkins-JavaDockerized.git'
      }
    }

    stage('Build JAR') {
      steps {
        bat 'mvn clean package -DskipTests'
      }
    }

    stage('Generate Dockerfile') {
      steps {
        script {
          writeFile file: 'Dockerfile', text: """
FROM eclipse-temurin:17-jdk-alpine
ARG JAR_FILE=target/*.jar
COPY \${JAR_FILE} app.jar
EXPOSE 7500
ENTRYPOINT ["java", "-jar", "/app.jar"]
"""
        }
      }
    }

    stage('Build Docker Image') {
      steps {
        bat 'docker build -t %IMAGE%:latest .'
      }
    }

    stage('Push to DockerHub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            bat 'docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%'
            bat 'docker push %IMAGE%:latest'
          }
        }
      }
    }

    stage('Run Container') {
      steps {
        bat 'docker rm -f javaapp || echo "No container to remove"'
        bat 'docker run -d -p 7500:8080 --name javaapp %IMAGE%:latest'
      }
    }
  }

  post {
    always {
      bat 'docker logout'
    }
  }
}
