pipeline {
    agent any
    // triggers{
    //     pollSCM('* * * * *')
    // }
    environment {
        AWS_ACCOUNT_ID = '296062588728'
        AWS_REGION = 'eu-central-1'
        ECR_REPO_NAME = 'acr-repo'
        IMAGE_TAG = 'latest'  // You can dynamically set the build version
        ECR_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}"
        EMAIL = "mounikapathuri30@gmail.com"
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $ECR_REPO_NAME .
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                sh '''
                docker tag $ECR_REPO_NAME:$IMAGE_TAG $ECR_URI
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_URI
                '''
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                docker pull $ECR_URI
                docker stop app || true
                docker rm app || true
                docker run -d --name app -p 8000:5000 $ECR_URI
                '''
            }
        }
    }

    post {
        success {
            mail to: "${EMAIL}",
                 subject: "Job '${env.JOB_NAME}' #${env.BUILD_NUMBER} Succeeded",
                 body: "Good news! The Jenkins job succeeded."
        }
        failure {
            mail to: "${EMAIL}",
                 subject: "Job '${env.JOB_NAME}' #${env.BUILD_NUMBER} Failed",
                 body: "Unfortunately, the Jenkins job failed. Please check the logs."
        }
    }
}
