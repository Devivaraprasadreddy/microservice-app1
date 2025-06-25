pipeline {
  agent any
  
  parameters {
        choice(name: 'BRANCH_NAME', choices: ['main'], description: 'Choose the branch to deploy')
    }

  environment {
    AWS_REGION = 'us-east-1'
    ECR_REPO = '084375571314.dkr.ecr.us-east-1.amazonaws.com/myapp'
    GIT_CREDENTIALS_ID = 'glpat--8zZqxEaFaUkUYjQ18ZB'
  }

  stages {
    stage('Checkout') {
        steps {
                script {
                    // Checkout the specified branch using Git credentials
                    checkout([$class: 'GitSCM', branches: [[name: "${params.BRANCH_NAME}"]],
                              userRemoteConfigs: [[url: 'https://gitlab.com/dsp9391/microservice-app11.git', credentialsId: "${env.GIT_CREDENTIALS_ID}"]]])
                }
            }
    }
  
    stage('Build with Maven') {
      steps {
        sh 'mvn clean package'
        echo "stage mvn done"
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t myapp .'
        echo "stage docker build done"
      }
    }
    /*stage('ECR Login') {
      steps {
        echo "stage entry for ecr done"
        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO'
        echo "stage  for ecr done"
      }
    }*/

    
  stage('Login to ECR') {
    steps {
      withCredentials([usernamePassword(credentialsId: 'aws-ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        sh '''
          aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
          aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
          aws configure set region $AWS_REGION
          
          aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
        '''
      }
    }
  }

    stage('Push to ECR') {
      steps {
        echo "stage  entry for Push to ECR done"
        sh '''
          docker tag myapp:latest $ECR_REPO:latest
          docker push $ECR_REPO:latest
        '''
        echo "stage  Push to ECR done"
      }
    }
  }
}
