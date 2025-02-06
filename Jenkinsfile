pipeline {
    agent {
        docker {
            image 'python:3.10-slim'  // Using the Python 3.10 slim image
        }
    }

    environment {
        EC2_HOST = credentials('EC2_HOST')  // EC2 Instance IP or Hostname
        EC2_USER = credentials('EC2_USER')  // EC2 Instance Username (e.g., ubuntu)
        SSH_KEY = credentials('SSH_KEY')    // SSH private key for EC2
    }

    stages {
        stage('Checkout Repository') {
            steps {
                checkout scm  // Checkout the latest code from the repository
            }
        }

        stage('Set Up Python Environment') {
            steps {
                sh '''
                    # Upgrade pip and install virtualenv locally
                    python3 -m pip install --upgrade pip
                    python3 -m pip install --user virtualenv

                    # Create and activate virtual environment
                    python3 -m virtualenv myenv
                    source myenv/bin/activate

                    # Install dependencies from requirements.txt
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                sh '''
                    source myenv/bin/activate
                    python3 train.py  # Train the model (replace with your actual script)
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                sh '''
                    # Save the SSH key as ec2_server.pem
                    echo "$SSH_KEY" | tr -d '\r' > ec2_server.pem
                    chmod 600 ec2_server.pem
                    
                    # Ensure the .ssh directory exists
                    mkdir -p ~/.ssh

                    # Automatically add EC2 host to known_hosts file
                    ssh-keyscan -H $EC2_HOST >> ~/.ssh/known_hosts

                    # Verify the SSH key and create directory if not exists
                    ssh -i ec2_server.pem $EC2_USER@$EC2_HOST '[ -d /home/$EC2_USER/models ] || sudo mkdir -p /home/$EC2_USER/models'

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
