pipeline {
    agent any

    parameters {
        choice(name: 'ACTIVE', choices: ['blue', 'green'], description: 'Current live environment')
    }

    environment {
        BLUE_IP = '172.31.1.168'
        GREEN_IP = '172.31.2.100'
        SSH_KEY = '/var/lib/jenkins/.ssh/sydney.pem'
        USER = 'ubuntu'
        LISTENER_ARN = 'arn:aws:elasticloadbalancing:ap-southeast-2:343218181062:listener/app/blue-green-alb/5f92535ce5cb10a4/7d1b940508ce786d'
        BLUE_TG_ARN = 'arn:aws:elasticloadbalancing:ap-southeast-2:343218181062:targetgroup/blue-tg/397ee940b08e0874'
        GREEN_TG_ARN = 'arn:aws:elasticloadbalancing:ap-southeast-2:343218181062:targetgroup/green-tg/f2012dda9992952b'
    }

    stages {
        stage('Pick Target') {
            steps {
                script {
                    if (params.ACTIVE == 'blue') {
                        env.TARGET_IP = env.GREEN_IP
                        env.TARGET_TG = env.GREEN_TG_ARN
                        env.PREV_TG = env.BLUE_TG_ARN
                    } else {
                        env.TARGET_IP = env.BLUE_IP
                        env.TARGET_TG = env.BLUE_TG_ARN
                        env.PREV_TG = env.GREEN_TG_ARN
                    }
                    echo "Deploying to $TARGET_IP"
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
                script {
                    try {
                        sh '''
                        for i in {1..5}; do
                            curl -f http://$TARGET_IP/health && exit 0
                            sleep 5
                        done
                        exit 1
                        '''
                        echo "Health check passed!"
                    } catch (Exception e) {
                        echo "Health check failed! Rolling back..."
                        sh "aws elbv2 modify-listener --listener-arn $LISTENER_ARN --default-actions Type=forward,TargetGroupArn=$PREV_TG"
                        error("Deployment failed, rolled back to previous environment")
                    }
                }
            }
        }

        stage('Switch ALB Traffic') {
            steps {
                sh "aws elbv2 modify-listener --listener-arn $LISTENER_ARN --default-actions Type=forward,TargetGroupArn=$TARGET_TG"
                echo "Traffic switched to $TARGET_IP"
            }
        }
    }
}
