pipeline {
  agent any

  environment {
    AWS_REGION      = 'us-east-2'
    ECR_REGISTRY    = '691652240094.dkr.ecr.us-east-2.amazonaws.com'
    ECR_REPO        = 'hello-world-app'
    IMAGE_TAG       = "${env.BUILD_NUMBER}"
    CLUSTER_NAME    = 'hello-world-cluster'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        sh """
          docker build --platform linux/amd64 -t ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG} ./app
        """
      }
    }

    stage('Push to ECR') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'Admin-IAM']]) {
          sh """
            aws ecr get-login-password --region ${AWS_REGION} | \
              docker login --username AWS --password-stdin ${ECR_REGISTRY}
            docker push ${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Deploy to EKS') {
      steps {
        withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                          credentialsId: 'Admin-IAM']]) {
          sh """
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
            kubectl set image deployment/hello-world hello-world=${ECR_REGISTRY}/${ECR_REPO}:${IMAGE_TAG}
            kubectl rollout status deployment/hello-world
          """
        }
      }
    }
  }

  post {
    success { echo 'Deployment successful!' }
    failure { echo 'Pipeline failed. Check logs above.' }
  }
}
