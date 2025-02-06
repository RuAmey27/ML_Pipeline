pipeline {
    agent {
        docker {
            image 'python:3.10-slim'
            args '--user root'  // Run the container as root
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
                    # Upgrade pip and install dependencies
                    pip install --upgrade pip
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
                    echo "$SSH_KEY" > ec2_server.pem
                    chmod 600 ec2_server.pem
                    
                    # Ensure model directory exists and is accessible
                    ssh -i ec2_server.pem $EC2_USER@$EC2_HOST "mkdir -p /home/$EC2_USER/models && chmod 755 /home/$EC2_USER/models"

                    # Copy the trained model to EC2
                    scp -i ec2_server.pem models/model_*.pkl $EC2_USER@$EC2_HOST:/home/$EC2_USER/models/
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
