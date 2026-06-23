pipeline {
    agent {
        label 'aws'
    }

    environment {
        GIT_REPO = 'https://github.com/divyashreej/Jenkinsandjava.git'
        AWS_REGION = 'ap-south-1'
        ECR_REPO_NAME = 'test-image'
        AWS_ACCOUNT_ID = '048051882772'
        ECR_REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}" 
        IMAGE_URI = "${ECR_REPO_URI}:${BUILD_NUMBER}"
    }

    stages {

        stage('Verify AWS Access') {
            steps {
                script {
                    sh '''
                        echo "Checking IAM Role access..."
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Install AWS CLI') {
            steps {
                script {
                    sh '''
                        set -e

                        if ! command -v aws >/dev/null 2>&1; then
                            echo "Installing AWS CLI..."

                            sudo apt update
                            sudo apt install -y unzip curl

                            curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

                            rm -rf aws
                            unzip -q awscliv2.zip

                            sudo ./aws/install --update
                        fi

                        aws --version
                    '''
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git url: "${GIT_REPO}", branch: 'main'
            }
        }

        stage('Build Java Application') {
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

                        aws ecr get-login-password --region ${AWS_REGION} | \
                        docker login \
                        --username AWS \
                        --password-stdin \
                        ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
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
}
