pipeline {
    agent {
        label 'aws'
    }

    environment {
        GIT_REPO = 'https://github.com/divyashreej/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'test-image'
        ECR_PUBLIC_REPO_URI = '048051882772.dkr.ecr.ap-south-1.amazonaws.com/test-image'
        //IMAGE_TAG = 'latest'
        AWS_ACCOUNT_ID = '0480-5188-2772'
        IMAGE_URI = "${ECR_PUBLIC_REPO_URI}:${BUILD_NUMBER}"
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
            mkdir -p /home/ubuntu/jenkins/.aws
            echo "[default]" > /home/ubuntu/jenkins/.aws/credentials
            echo "aws_access_key_id=AKIAQWMA5RMKCS7N36H7" >> /home/ubuntu/jenkins/.aws/credentials
            echo "aws_secret_access_key=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" >> /home/ubuntu/jenkins/.aws/credentials
            chown -R ubuntu:ubuntu /home/ubuntu/jenkins/.aws
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
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 048051882772.dkr.ecr.ap-south-1.amazonaws.com/test-image
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        echo "Building Docker image..."
                        sudo docker build -t ${IMAGE_URI} .
                    '''
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                script {
                    sh '''
                        echo "Pushing Docker image to ECR..."
                        sudo docker push ${IMAGE_URI}
                            '''
                        }
                    }
        }
    }
}    
