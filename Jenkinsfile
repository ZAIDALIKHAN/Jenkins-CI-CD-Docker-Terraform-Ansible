pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REPO = '069380454032.dkr.ecr.us-east-1.amazonaws.com/myapp'
        IMAGE_NAME = 'myapp'
        IMAGE_TAG = 'latest'
        APP_SERVER = 'ubuntu@52.207.33.241'
        PEM_PATH = '/var/lib/jenkins/keys/Pair.pem'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ZAIDALIKHAN/Jenkins-CI-CD-Docker-Terraform-Ansible.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Login to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REPO
                '''
            }
        }

        stage('Tag & Push to ECR') {
            steps {
                sh '''
                docker tag $IMAGE_NAME:latest $ECR_REPO:$IMAGE_TAG
                docker push $ECR_REPO:$IMAGE_TAG
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no -i $PEM_PATH $APP_SERVER '
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REPO &&
                docker pull $ECR_REPO:$IMAGE_TAG &&
                docker stop myapp || true &&
                docker rm myapp || true &&
                docker run -d -p 80:8080 --name myapp $ECR_REPO:$IMAGE_TAG
                '
                '''
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
