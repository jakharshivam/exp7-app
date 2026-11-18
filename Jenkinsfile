pipeline {
  agent any
  environment { IMAGE = 'exp7-app' }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }
    stage('Build & Push Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'docker build -t $DH_USER/$IMAGE:$BUILD_NUMBER .'
          sh 'echo $DH_PASS | docker login -u $DH_USER --password-stdin'
          sh 'docker push $DH_USER/$IMAGE:$BUILD_NUMBER'
        }
      }
    }
    stage('Deploy to Kubernetes') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
          sh 'sed "s|IMAGE_PLACEHOLDER|$DH_USER/$IMAGE:$BUILD_NUMBER|" deployment.yaml | kubectl apply -f -'
          sh 'kubectl rollout status deployment/exp7-app --timeout=120s'
        }
      }
    }
  }
}
