pipeline {
    agent any

    parameters {
        choice(name: 'ACTIVE', choices: ['blue', 'green'], description: 'Current live env')
    }

    environment {
    BLUE_IP = '172.31.1.168'
    GREEN_IP = '172.31.2.100'
    SSH_KEY = '/var/lib/jenkins/.ssh/sydney.pem'
    USER = 'ubuntu'
}

    stages {

        stage('Pick Target') {
            steps {
                script {
                    if (params.ACTIVE == 'blue') {
                        env.TARGET_IP = env.GREEN_IP
                    } else {
                        env.TARGET_IP = env.BLUE_IP
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                scp -i $SSH_KEY -o StrictHostKeyChecking=no app/index.html $USER@$TARGET_IP:/tmp/
                scp -i $SSH_KEY -o StrictHostKeyChecking=no app/health $USER@$TARGET_IP:/tmp/
                ssh -i $SSH_KEY -o StrictHostKeyChecking=no $USER@$TARGET_IP "
                    sudo cp /tmp/index.html /var/www/html/index.html &&
                    sudo cp /tmp/health /var/www/html/health &&
                    sudo systemctl restart nginx
                "
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                curl -f http://$TARGET_IP/health
                '''
            }
        }

    }
}
