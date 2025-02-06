pipeline {
    agent {
        docker {
            image 'python:3.10-slim'  // Using Python 3.10 slim Docker image
        }
    }

    environment {
        EC2_HOST = credentials('EC2_HOST')  // EC2 Instance IP or Hostname
        EC2_USER = credentials('EC2_USER')  // EC2 Instance Username (e.g., ubuntu)
        SSH_KEY = credentials('SSH_KEY')    // SSH private key (ec2_server.pem)
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm  // Checkout the latest code from the repository
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    # Upgrade pip and install dependencies globally inside the container
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                sh '''
                    # Run the training script inside the container
                    python3 train.py  # Train the model (replace with your actual script)
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                sh '''
                    # Save the SSH key as ec2_server.pem
                    echo "$SSH_KEY" > ec2_server.pem
                    chmod 600 ec2_server.pem
                    
                    # Create the models directory on the EC2 instance if it does not exist
                    ssh -i ec2_server.pem $EC2_USER@$EC2_HOST '[ -d "/home/$EC2_USER/models" ] || sudo mkdir -p /home/$EC2_USER/models'

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
