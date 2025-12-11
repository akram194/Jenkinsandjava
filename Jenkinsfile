pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/akram194/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1b'
        ECR_REPO_NAME = 'my-ecr-private'
        ECR_PUBLIC_REPO_URI = '797622874630.dkr.ecr.ap-south-1.amazonaws.com/my-ecr-private'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '797622874630'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
  
    }

    stages {
        stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e
                        echo "Installing AWS CLI..."
                        sudo apt update && sudo apt install -y unzip curl

                        curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
                        rm -rf aws
                        unzip -q awscliv2.zip
                        sudo ./aws/install --update
                        aws --version
                    '''
                }
            }
        }

             stage('Configure AWS Credentials') {
    steps {
        script {
            sh '''
            echo "Setting up AWS credentials for Jenkins..."
            mkdir -p /var/lib/jenkins/.aws
            echo "[default]" > /var/lib/jenkins/.aws/credentials
            echo "aws_access_key_id=AKIA3TNQHGIDB3EKW64R" >> /var/lib/jenkins/.aws/credentials
            echo "aws_secret_access_key=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" >> /var/lib/jenkins/.aws/credentials
            chown -R jenkins:jenkins /var/lib/jenkins/.aws
            '''
        }
    }
}


        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build') {
            steps {
                script {
                    sh '''
                        echo "Building Java application..."
                        mvn clean -B -Denforcer.skip=true package
                    '''
                }
            }
        }

        stage('Login to AWS ECR') {
            steps {
                script {
                    sh '''
                        echo "Logging into AWS ECR..."
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 797622874630.dkr.ecr.ap-south-1.amazonaws.com
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        docker push ${IMAGE_URI}
                    '''
                }
            }
        }

    }
    
    post {
        success {
            echo "Docker image pushed to ECR successfully and deployed."
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
