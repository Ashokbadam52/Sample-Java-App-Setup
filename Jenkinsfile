pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "java-app"
    SONARQUBE = "sonar" // Jenkins Sonar config name
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Ashokbadam52/Sample-Java-App-Setup.git'
      }
    }

    stage('Build') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Code Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE}") {
          sh 'mvn sonar:sonar'
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${DOCKER_IMAGE}:latest .'
      }
    }

    stage('K8s Deploy') {
      steps {
        sh 'kubectl apply -f k8s-deployment.yaml'
      }
    }
  }

  post {
    success {
      echo "Pipeline executed successfully!"
    }
    failure {
      echo "Pipeline failed!"
    }
  }
}
