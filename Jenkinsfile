pipeline {
    agent any
    environment {
        AWS_REGION     = 'eu-north-1'
        AWS_ACCOUNT_ID = '208731215970'
        ECR_REPOSITORY = 'image-regi'
        IMAGE_NAME     = 'cloud-image'
        ECR_REGISTRY   = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        ECR_IMAGE      = "${ECR_REGISTRY}/${ECR_REPOSITORY}:latest"
        DOCKER_CONFIG  = "${WORKSPACE}/.docker"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build and Test') {
            steps { sh 'python3 -m py_compile app.py' }
        }
        stage('Docker Build') {
            steps { sh 'docker build -t ${IMAGE_NAME}:latest .' }
        }
        stage('ECR Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-exp5-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'docker run --rm -e AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY amazon/aws-cli ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}'
                }
            }
        }
        stage('Docker Tag') {
            steps { sh 'docker tag ${IMAGE_NAME}:latest ${ECR_IMAGE}' }
        }
        stage('Push to ECR') {
            steps { sh 'docker push ${ECR_IMAGE}' }
        }
    }
    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
    }
}