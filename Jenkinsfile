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

        stage('Build and Test') {
            steps {
                dir('bookmyshow-app') {
                    sh '''
                    npm install --legacy-peer-deps
                    CI=false npm test -- --watchAll=false || true
                    '''
                }
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

        stage('Push Docker Image to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                docker push $ECR_URI:$IMAGE_TAG
                docker push $ECR_URI:latest
                '''
            }
        }

        stage('Deploy to EKS using Ansible') {
            steps {
                dir('ansible') {
                    sh '''
                    ansible-playbook -i hosts deploy.yml --extra-vars "image=$ECR_URI:$IMAGE_TAG"
                    '''
                }
            }
        }

        stage('Validate Deployment') {
            steps {
                sh '''
                kubectl get pods -n staging
                kubectl get pods -n production
                kubectl get svc -n staging
                kubectl get svc -n production
                '''
            }
        }
    }
}
