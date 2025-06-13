pipeline {
  agent any
  environment {
    IMAGE = 'anandhuraj111/containerized-java'
    DOCKER_CREDS = credentials('docker-id') // Make sure this ID exists in Jenkins
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
        script {
          docker.build(IMAGE)
        }
      }
    }
    stage('Push to DockerHub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'docker-id', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            bat 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
            bat 'docker push $IMAGE:latest'
            bat 'docker tag $IMAGE:latest $IMAGE:latest'
          }
        }
      }
    }
    stage('Run Container') {
      steps {
        bat 'docker rm -f javaapp || true'
        bat 'docker run -d -p 7500:8080 --name javaapp $IMAGE:latest'
      }
    }
  }
  post {
    always {
      bat 'docker logout'
    }
  }
}
