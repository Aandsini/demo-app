pipeline {
  agent any

  environment {
    IMAGE = "capeahhhhhhhhh/demo-app"
    TAG = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${IMAGE}:${TAG} ."
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
          sh '''
            echo $PASS | docker login -u $USER --password-stdin
            docker push ${IMAGE}:${TAG}
          '''
        }
      }
    }

    stage('Deploy to Kubernetes via Helm') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONF')]) {
          sh '''
            export KUBECONFIG=$KUBECONF
            helm upgrade --install demo-app helm-chart --set image.repository=${IMAGE} --set image.tag=${TAG}
          '''
        }
      }
    }

    stage('Verify Deployment') {
      steps {
        withCredentials([file(credentialsId: 'kubeconfig-file', variable: 'KUBECONF')]) {
          sh '''
            export KUBECONFIG=$KUBECONF
            kubectl get pods
          '''
        }
      }
    }
  }
}
