pipeline {
    agent {
        docker {
            image 'python:3.10-slim'
           args '--user root -t -d --entrypoint /bin/sh'  // Run as root
        }
    }

    environment {
        EC2_HOST = credentials('EC2_HOST')
        EC2_USER = credentials('EC2_USER')
        SSH_KEY = credentials('SSH_KEY')
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    apt-get update && apt-get install -y openssh-client python3-pip
                    
                    # Ensure pip is installed and upgraded
                    python3 -m ensurepip
                    python3 -m pip install --upgrade pip
                    
                    # Install required dependencies
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                sh '''
                    python3 train.py
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                sh '''
                    # Store the SSH key securely
                    echo "$SSH_KEY" > ec2_server.pem
                    chmod 600 ec2_server.pem
                    
                    # Ensure the model directory exists on EC2
                    ssh -o StrictHostKeyChecking=no -i ec2_server.pem $EC2_USER@$EC2_HOST "mkdir -p /home/$EC2_USER/models && chmod 755 /home/$EC2_USER/models"

                    # Copy the trained model to EC2
                    scp -o StrictHostKeyChecking=no -i ec2_server.pem models/model_*.pkl $EC2_USER@$EC2_HOST:/home/$EC2_USER/models/
                '''
            }
        }
    }

    post {
        success {
            echo 'Model retrained and deployed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
