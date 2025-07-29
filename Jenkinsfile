pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "java-app"
    SONARQUBE = "sonar" // This is the ID from Manage Jenkins > Configure System
    SONAR_AUTH = credentials('Sonarqube-token') // The token stored in Jenkins credentials
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
          sh 'mvn sonar:sonar -Dsonar.login=${SONAR_AUTH}'
        }
      }
    }

    // Optional: Wait for SonarQube Quality Gate
    /*
    stage('Quality Gate') {
      steps {
        timeout(time: 2, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    */

    stage('Docker Build') {
      steps {
        sh 'docker build -t "${DOCKER_IMAGE}:latest" .'
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
      echo "✅ Pipeline executed successfully!"
    }
    failure {
      echo "❌ Pipeline failed!"
    }
  }
}
