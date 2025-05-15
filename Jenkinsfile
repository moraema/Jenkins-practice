pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    environment {
        EC2_USER = 'ubuntu'
        SSH_KEY = credentials('ssh-key-ec2')
        DEV_IP = '34.197.126.56'
        PROD_IP = '34.197.126.56'
        REMOTE_PATH = '/home/ubuntu/Jenkins-practice'
    }

    stages {
        stage('Detect Branch') {
            steps {
                script {
                    env.ACTUAL_BRANCH = env.BRANCH_NAME ?: 'main'
                    echo " Rama activa: ${env.ACTUAL_BRANCH}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def ip = env.ACTUAL_BRANCH == 'develop' ? DEV_IP :
                             env.ACTUAL_BRANCH == 'main'    ? PROD_IP : null

                    def pm2_name = "${env.ACTUAL_BRANCH}-health"

                    if (ip == null) {
                        error "Branch ${env.ACTUAL_BRANCH} no está configurada para despliegue."
                    }

                    sh """
                    ssh -i $SSH_KEY -o StrictHostKeyChecking=no $EC2_USER@$ip '
                        echo "Actualizando sistema..."
                        sudo apt-get update -y &&
                        sudo apt-get upgrade -y

                        echo "Verificando Node.js..."
                        if ! command -v node > /dev/null; then
                            curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
                            sudo apt-get install -y nodejs
                        fi

                        echo "Verificando PM2..."
                        if ! command -v pm2 > /dev/null; then
                            sudo npm install -g pm2
                        fi

                        echo "Verificando carpeta de app..."
                        if [ ! -d "$REMOTE_PATH/.git" ]; then
                            git clone https://github.com/roberto14118927/node-healthcheck.git $REMOTE_PATH
                        fi

                        echo "Pull y deploy..."
                        cd $REMOTE_PATH &&
                        git pull origin ${env.ACTUAL_BRANCH} &&
                        npm ci &&
                        pm2 restart ${pm2_name} || pm2 start server.js --name ${pm2_name}
                    '
                    """
                }
            }
        }
    }
}