pipeline {
    agent any  // Run on any available Jenkins agent

    environment {
        // Credentials for EC2 (Make sure these credentials are added in Jenkins credentials)
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
                // Create a virtual environment and install dependencies using bash
                sh '''
                    # Use bash to run the following commands
                    bash -c "python3 -m venv myenv"
                    bash -c "source myenv/bin/activate"
                    bash -c "pip install --upgrade pip"
                    bash -c "pip install -r requirements.txt"
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                // Retrain the model using the Python training script
                sh '''
                    bash -c "source myenv/bin/activate"
                    bash -c "python3 train.py"  // Retrain the model (replace with your actual script)
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                // Deploy the trained model to the target EC2 instance
                sh '''
                    bash -c "echo '$SSH_KEY' | tr -d '\r' > Flaskapp.pem"
                    bash -c "chmod 600 Flaskapp.pem"
                    
                    bash -c "mkdir -p ~/.ssh"
                    bash -c "ssh-keyscan -H $EC2_HOST >> ~/.ssh/known_hosts"
                    
                    bash -c "ssh -i Flaskapp.pem $EC2_USER@$EC2_HOST '[ -d /home/$EC2_USER/models ] || sudo mkdir -p /home/$EC2_USER/models'"
                    bash -c "scp -i Flaskapp.pem models/model_*.pkl $EC2_USER@$EC2_HOST:/home/$EC2_USER/models/"
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
