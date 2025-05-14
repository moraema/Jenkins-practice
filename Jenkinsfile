pipeline {
    agent any
    
    environment {
       
        SSH_KEY = credentials('ssh-key-ec2')
    }
    
    stages {
        stage('Determine Environment') {
            steps {
                script {
                   
                    def branchName = env.GIT_BRANCH
                    
                    echo "Rama detectada desde Jenkins: ${branchName}"
                    
                
                    if (branchName.startsWith('origin/')) {
                        branchName = branchName.substring('origin/'.length())
                    }
                    
                    env.GIT_BRANCH_CLEAN = branchName
                    echo "Rama normalizada: ${env.GIT_BRANCH_CLEAN}"
                    
                    
                    if (env.GIT_BRANCH_CLEAN == 'main') {
                        env.DEPLOY_ENV = 'PROD'
                        env.NODE_ENV = 'production'
                        env.EC2_USER = 'ubuntu'
                        env.EC2_IP = '34.197.126.56'
                        env.REMOTE_PATH = '/home/ubuntu/Jenkins-practice'
                        env.APP_NAME = 'health-api'
                    } else if (env.GIT_BRANCH_CLEAN == 'dev') {
                        env.DEPLOY_ENV = 'DEV'
                        env.NODE_ENV = 'development'
                        env.EC2_USER = 'ubuntu'
                        env.EC2_IP = '192.168.1.10'
                        env.REMOTE_PATH = '/home/ubuntu/Jenkins-practice'
                        env.APP_NAME = 'health-api'
                    } else if (env.GIT_BRANCH_CLEAN == 'QA') {
                        env.DEPLOY_ENV = 'QA'
                        env.NODE_ENV = 'testing'
                        env.EC2_USER = 'ubuntu'
                        env.EC2_IP = '192.168.1.20'
                        env.REMOTE_PATH = '/home/ubuntu/Jenkins-practice'
                        env.APP_NAME = 'health-api'
                    } else {
                        error "No se desplegará desde la rama ${env.GIT_BRANCH_CLEAN}. Solo se permiten las ramas main, dev y QA."
                    }
                    
                    echo "Desplegando en entorno ${env.DEPLOY_ENV} desde la rama ${env.GIT_BRANCH_CLEAN}"
                }
            }
        }
        
        stage('Checkout') {
            steps {

                git branch: env.GIT_BRANCH_CLEAN, url: 'https://github.com/moraema/Jenkins-practice.git'
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
                    mkdir -p $REMOTE_PATH &&
                    cd $REMOTE_PATH &&
                    git fetch --all &&
                    git checkout ${env.GIT_BRANCH_CLEAN} &&
                    git pull origin ${env.GIT_BRANCH_CLEAN} &&
                    export NODE_ENV=${env.NODE_ENV} &&
                    npm ci &&
                    pm2 restart ${env.APP_NAME} || pm2 start server.js --name ${env.APP_NAME}
                '
                """
            }
        }
    }
    
    post {
        success {
            echo "Despliegue exitoso en entorno ${env.DEPLOY_ENV}"
        }
        failure {
            echo "Fallo en el despliegue para entorno ${env.DEPLOY_ENV}"
        }
    }
}