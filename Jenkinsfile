pipeline {
    agent { label 'docker-agent-label' }
 
    environment {
        AWS_ACCOUNT_ID = '571600845308'  // Replace with actual AWS Account ID
        AWS_REGION = 'us-east-1'         // Update region if needed
        IMAGE_NAME = 'todo-app'
        REPO_NAME = 'todo-ecr-repo'
        EC2_USER = 'ubuntu'
EC2_HOST = '52.23.170.248'
        SSH_KEY = 'lms-jenkins-keypair.pem'
    }
 
    stages {
        stage('Clone Repository') {
            steps {
git 'https://github.com/Gayathri-Musham-07/to-do-list-practise.git'
            }
        }
 
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
 
        stage('Run Pylint Checks') {
            steps {
                sh 'pip install pylint'
                sh 'pylint --fail-under=8 $(find . -name "*.py")'  // Scan all Python files
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$BUILD_NUMBER .
docker tag $IMAGE_NAME:$BUILD_NUMBER $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_NUMBER
                '''
            }
        }
 
        stage('Push to AWS ECR') {
            steps {
                sh '''
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_NUMBER
                '''
            }
        }
 
        stage('Start AWS ECR Image Scan') {
            steps {
                sh '''
                aws ecr start-image-scan --repository-name $REPO_NAME --image-id imageTag=$BUILD_NUMBER
                '''
            }
        }
 
        stage('Check AWS ECR Scan Results') {
            steps {
                sh '''
                aws ecr describe-image-scan-findings --repository-name $REPO_NAME --image-id imageTag=$BUILD_NUMBER
                '''
            }
        }
 
        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh-credential']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no -i $SSH_KEY $EC2_USER@$EC2_HOST << EOF
docker login -u AWS -p $(aws ecr get-login-password --region $AWS_REGION) $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
docker pull $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_NUMBER
                    docker stop todo-container || true
                    docker rm todo-container || true
docker run -d --name todo-container -p 80:5000 $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_NUMBER
                    EOF
                    '''
                }
            }
        }
    }
}
