pipeline {
  agent any
  environment {
    IMAGE = 'shaikkhizar/springboot-jenkins'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Test') {
      steps { sh 'mvn -q -DskipTests=false clean verify' }
      post { always { junit '**/target/surefire-reports/*.xml' } }
    }
    stage('Package') {
      steps { sh 'mvn -q -DskipTests package' }
    }
    stage('Docker Build') {
      steps { sh "docker build -t ${IMAGE}:${env.BUILD_NUMBER} -t ${IMAGE}:latest ." }
    }
    stage('Docker Push') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh 'echo $PASS | docker login -u $USER --password-stdin'
          sh "docker push ${IMAGE}:${env.BUILD_NUMBER}"
          sh "docker push ${IMAGE}:latest"
        }
      }
    }
    stage('Deploy (Local Docker)') {
      steps {
        sh """
          docker rm -f springboot-app || true
          docker run -d --name springboot-app -p 8081:8080 ${IMAGE}:latest
        """
      }
    }
  }
  post {
    success {
      echo "Deployed → http://<VM-IP>:8081/hello"
    }
  }
}
