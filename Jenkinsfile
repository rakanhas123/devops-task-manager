pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        AWS_ACCOUNT_ID = '717958588488'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        BACKEND_REPO = 'task-manager-backend'
        FRONTEND_REPO = 'task-manager-frontend'
        IMAGE_TAG = "v${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Images') {
            steps {
                bat 'docker build -t devops-task-manager-backend:%IMAGE_TAG% ./backend'
                bat 'docker build -t devops-task-manager-frontend:%IMAGE_TAG% ./frontend'
            }
        }

        stage('Login to ECR') {
            steps {
                bat 'aws ecr get-login-password --region %AWS_REGION% | docker login --username AWS --password-stdin %ECR_REGISTRY%'
            }
        }

        stage('Tag Images') {
            steps {
                bat 'docker tag devops-task-manager-backend:%IMAGE_TAG% %ECR_REGISTRY%/%BACKEND_REPO%:%IMAGE_TAG%'
                bat 'docker tag devops-task-manager-frontend:%IMAGE_TAG% %ECR_REGISTRY%/%FRONTEND_REPO%:%IMAGE_TAG%'
            }
        }

        stage('Push Images to ECR') {
            steps {
                bat 'docker push %ECR_REGISTRY%/%BACKEND_REPO%:%IMAGE_TAG%'
                bat 'docker push %ECR_REGISTRY%/%FRONTEND_REPO%:%IMAGE_TAG%'
            }
        }

        stage('Update Kubernetes') {
            steps {
                bat 'kubectl set image deployment/backend backend=%ECR_REGISTRY%/%BACKEND_REPO%:%IMAGE_TAG%'
                bat 'kubectl set image deployment/frontend frontend=%ECR_REGISTRY%/%FRONTEND_REPO%:%IMAGE_TAG%'
            }
        }

        stage('Check Kubernetes Status') {
            steps {
                bat 'kubectl rollout status deployment/backend'
                bat 'kubectl rollout status deployment/frontend'
                bat 'kubectl get pods'
                bat 'kubectl get svc'
                bat 'kubectl get ingress'
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs.'
        }
    }
}