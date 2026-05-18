pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ACCOUNT_ID = '653136267460'
        ECR_REPO = 'bookmyshow'
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Vamsikrushna18/BookMyShow.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                    docker build -t bookmyshow:$IMAGE_TAG .
                    docker tag bookmyshow:$IMAGE_TAG $ECR_URI:$IMAGE_TAG
                    docker tag bookmyshow:$IMAGE_TAG $ECR_URI:latest
                    '''
                }
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh '''
                docker push $ECR_URI:$IMAGE_TAG
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                kubectl set image deployment/bms-app bms-container=$ECR_URI:$IMAGE_TAG
                kubectl rollout status deployment/bms-app
                '''
            }
        }
    }
}
