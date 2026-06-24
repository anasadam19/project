pipeline {
    agent any

    environment {
        AWS_REGION = "ap-south-1"
        ECR_REPO = "account-id.dkr.ecr.ap-south-1.amazonaws.com/shopping-app"
        IMAGE_NAME = "shopping-app"
        EC2_HOST = "ec2-user@<EC2_PUBLIC_IP>"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/shopping-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION | \
                docker login --username AWS --password-stdin $ECR_REPO
                docker tag $IMAGE_NAME:latest $ECR_REPO:latest
                docker push $ECR_REPO:latest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sh '''
                ssh -o StrictHostKeyChecking=no $EC2_HOST \
                "docker pull $ECR_REPO:latest && \
                 docker stop shopping-app || true && \
                 docker rm shopping-app || true && \
                 docker run -d -p 80:8080 --name shopping-app $ECR_REPO:latest"
                '''
            }
        }

        stage('Validation') {
            steps {
                sh 'curl -f http://<EC2_PUBLIC_IP>'
            }
        }
    }
}

