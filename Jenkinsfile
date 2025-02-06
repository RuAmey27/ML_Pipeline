pipeline {
    agent {
        docker {
            image 'python:3.10-slim'  // Use Python 3.10 image
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
                // Create a virtual environment and install dependencies inside Docker container
                sh '''
                    python3 -m venv myenv
                    source myenv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Retrain the Model') {
            steps {
                // Retrain the model inside the Docker container
                sh '''
                    source myenv/bin/activate
                    python3 train.py  // Retrain the model (replace with your actual script)
                '''
            }
        }

        stage('Deploy Model to EC2') {
            steps {
                // Deploy the trained model to the target EC2 instance
                sh '''
                    echo "$SSH_KEY" | tr -d '\r' > Flaskapp.pem
                    chmod 600 Flaskapp.pem
                    mkdir -p ~/.ssh
                    ssh-keyscan -H $EC2_HOST >> ~/.ssh/known_hosts
                    ssh -i Flaskapp.pem $EC2_USER@$EC2_HOST '[ -d /home/$EC2_USER/models ] || sudo mkdir -p /home/$EC2_USER/models'
                    scp -i Flaskapp.pem models/model_*.pkl $EC2_USER@$EC2_HOST:/home/$EC2_USER/models/
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
