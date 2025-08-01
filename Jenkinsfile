pipeline{
  agent any

  environment{
    ECR_REGISTRY = '745490702538.dkr.ecr.us-east-1.amazonaws.com'
    REPOSITORY = 'eschonaufinalproject'
    IMAGE_NAME = 'finalproject'
    FULL_IMAGE_NAME = "${ECR_REGISTRY}/${REPOSITORY}:${IMAGE_NAME}"
    EC2_HOST = 'ubuntu@44.204.38.8'
  }
  stages{
    stage('Checkout Out'){
      steps{
        git branch: 'main', url: 'https://github.com/Emmy-S-300/4299-Final-project.git'
      }
    }
    stage('Login, build, and push to ECR'){
      steps{
        withAWS(region: 'us-east-1', credentials: "aws-creds"){
          sh """
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY}
          """
        }
      }
    }
    stage('Build docker image'){
      steps{
        sh """
        docker build -t ${IMAGE_NAME} .
        docker tag ${IMAGE_NAME} ${FULL_IMAGE_NAME}
        """
      }
    }
    stage('Pushing image to ECR'){
      steps{
        sh """
        docker push ${FULL_IMAGE_NAME}
        """
      }
    }
    stage('Deploy to EC2'){
      steps{
        sshagent(['ec2-ssh-key']) {
          sh """
            ssh -o StrictHostKeyChecking=no ${EC2_HOST} '
              cd /home/ubuntu/4299-Final-project &&
              aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ECR_REGISTRY} &&
              docker-compose down -v &&
              docker-compose pull &&
              docker-compose up -d --build
            '
        """
        }
      }
    }
  }
}
