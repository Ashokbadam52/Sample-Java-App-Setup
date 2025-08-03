pipeline {
  agent any

  environment {
    DOCKER_IMAGE = "ashokdevops582/java-ap"
    SONARQUBE = "sonar" // This must match Jenkins Global Tool Configuration
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
        SONAR_AUTH = credentials('Sonarqube-token')
      }
      steps {
        withSonarQubeEnv("${SONARQUBE}") {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH'
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          docker build -t $DOCKER_IMAGE:latest .
          docker tag $DOCKER_IMAGE:latest $DOCKER_IMAGE:${BUILD_NUMBER}
        '''
      }
    }

    stage('DockerHub Login') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
          '''
        }
      }
    }

    stage('Push Image') {
      steps {
        sh 'docker push $DOCKER_IMAGE:${BUILD_NUMBER}'
      }
    }
    stage('Deploy to EC2') {
  steps {
    sshagent (credentials: ['ec2-ssh']) {
      sh '''
        ssh -o StrictHostKeyChecking=no ec2-user@52.91.78.148 << EOF
          sudo docker pull ashokdevops582/java-app:${BUILD_NUMBER}
          sudo docker stop java-app || true
          sudo docker rm java-app || true
          sudo docker run -d --name java-app -p 8080:8080 ashokdevops582/java-app:${BUILD_NUMBER}
        EOF
      '''
    }
  }
}

    // Uncomment below stages once KUBECONFIG is properly setup
    /*
    stage('K8s Deploy') {
      steps {
        sh '''
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          kubectl apply -f k8s-deployment.yaml
        '''
      }
    }

    stage('Verify K8s Deployment') {
      steps {
        sh '''
          export KUBECONFIG=/var/lib/jenkins/.kube/config
          kubectl get pods
          kubectl get svc
        '''
      }
    }
    */
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
