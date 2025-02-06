pipeline {
    agent any  // Run on any available Jenkins agent

    environment {
        // Credentials for EC2 (Make sure these credentials are added in Jenkins credentials)
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

        stage('Set Up Python Environment') {
            steps {
                // Create a virtual environment and install dependencies
                sh '''
                    # Create a virtual environment
                    python3 -m venv myenv
                    
                    # Activate the virtual environment
                    source myenv/bin/activate
                    
                    # Install dependencies from requirements.txt
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                // Retrain the model using the Python training script
                sh '''
                    source myenv/bin/activate
                    python3 train.py  // Train the model (replace with your actual script)
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                // Deploy the trained model to the target EC2 instance
                sh '''
                    # Save the SSH key and fix any issues with carriage returns
                    echo "$SSH_KEY" | tr -d '\r' > ec2_server.pem
                    chmod 600 ec2_server.pem
                    
                    # Ensure the models directory exists on the EC2 instance
                    ssh -i ec2_server.pem $EC2_USER@$EC2_HOST '[ -d "/home/$EC2_USER/models" ] || sudo mkdir -p /home/$EC2_USER/models'
                    
                    # Copy the trained model file to EC2
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
