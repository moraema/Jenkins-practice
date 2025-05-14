pipeline {
    agent any

    environment {
        SSH_KEY = credentials('ssh-key-ec2')
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/moraema/Jenkins-practice.git', branch: "${env.BRANCH_NAME}"
            }
        }

        stage('Build') {
            steps {
                sh 'rm -rf node_modules'
                sh 'npm ci'
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'develop'
                    branch 'qa'
                    branch 'main'
                }
            }
            steps {
                script {
                    def config = [
                        develop: [
                            NODE_ENV: 'development',
                            EC2_IP: '3.85.100.201',
                            REMOTE_PATH: '/home/ubuntu/Jenkins-practice',
                            APP_NAME: 'health-api-dev'
                        ],
                        qa: [
                            NODE_ENV: 'qa',
                            EC2_IP: '3.92.110.101',
                            REMOTE_PATH: '/home/ubuntu/Jenkins-practice',
                            APP_NAME: 'health-api-qa'
                        ],
                        main: [
                            NODE_ENV: 'production',
                            EC2_IP: '34.197.126.56',
                            REMOTE_PATH: '/home/ubuntu/Jenkins-practice',
                            APP_NAME: 'health-api'
                        ]
                    ][env.BRANCH_NAME]

                    if (config == null) {
                        error "No deployment config found for branch: ${env.BRANCH_NAME}"
                    }

                    sh """
                        ssh -i $SSH_KEY -o StrictHostKeyChecking=no ubuntu@${config.EC2_IP} '
                            export NODE_ENV=${config.NODE_ENV} &&
                            cd ${config.REMOTE_PATH} &&
                            git pull origin ${env.BRANCH_NAME} &&
                            npm ci &&
                            pm2 restart ${config.APP_NAME} || pm2 start server.js --name ${config.APP_NAME}
                        '
                    """
                }
            }
        }
    }
}