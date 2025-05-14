pipeline {
    agent any

    environment {
        NODE_ENV = 'production'
        EC2_USER = 'ubuntu'
        EC2_IP = '34.197.126.56'
        REMOTE_PATH = '/home/ubuntu/Jenkins-practice'
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/moraema/Jenkins-practice.git'
            }
        }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'    
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            steps {
                sh """
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$EC2_IP '
                    cd $REMOTE_PATH &&
                    git pull origin main &&
                    npm ci &&
                    pm2 restart health-api || pm2 start server.js --name health-api
                '
                """
            }
        }
    }
}
