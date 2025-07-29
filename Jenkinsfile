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
      environment {
        SONAR_AUTH = credentials('Sonarqube-token') // Use your Jenkins credential ID
      }
      steps {
        withSonarQubeEnv("${SONARQUBE}") {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH'
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t $DOCKER_IMAGE:latest .'
      }
    }

    stage('K8s Deploy') {
      steps {
        sh 'export KUBECONFIG=/var/lib/jenkins/.kube/config && kubectl apply -f k8s-deployment.yaml'
      }
    }

    stage('Verify K8s Deployment') {
      steps {
        withEnv(["KUBECONFIG=/var/lib/jenkins/.kube/config"]) {
          sh 'kubectl get pods'
          sh 'kubectl get svc'
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline executed successfully!"
    }
    failure {
      echo "❌ Pipeline failed. Check the logs above."
    }
  }
}
