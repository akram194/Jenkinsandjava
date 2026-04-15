pipeline {
    agent any

    environment {
        GIT_REPO = 'https://github.com/akram194/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'jenkinsecr'
        ECR_PUBLIC_REPO_URI = '499193102247.dkr.ecr.ap-south-1.amazonaws.com/jenkinsecr'
        IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '499193102247'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${IMAGE_TAG}"
        EKS_CLUSTER = 'my-eks-cluster2'
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
                        echo "aws_access_key_id=AKIAXIOR2H6TWLYDTZG5" >> /var/lib/jenkins/.aws/credentials
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
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 499193102247.dkr.ecr.ap-south-1.amazonaws.com
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

        stage('Deploy to EKS') {
            steps {
                script {
                    sh '''
                        echo "Updating kubeconfig..."
                        mkdir -p /var/lib/jenkins/.kube
                        aws eks update-kubeconfig --region $AWS_REGION --name $EKS_CLUSTER --kubeconfig /var/lib/jenkins/.kube/config
                        export KUBECONFIG=/var/lib/jenkins/.kube/config

                        echo "Applying Kubernetes manifests..."
                        kubectl apply -f Jenkinsandjava/deploymentjava.yaml --validate=false
                        kubectl apply -f Jenkinsandjava/servicelb.yaml --validate=false
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
