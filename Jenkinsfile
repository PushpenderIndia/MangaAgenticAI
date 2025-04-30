pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', credentialsId: 'be931eed-297a-4c57-9706-565d76161ee0', url: 'https://github.com/WitesoAI/Comify'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh 'sudo apt install python3-pip -y && pip3 install virtualenv'
                    sh '''
                    chmod +x envsetup.sh
                    ./envsetup.sh
                    '''
                    sh 'sudo apt remove chromium-browser -y'
                }
            }
        }

        stage('Install Redis') {
            steps {
                script {
                    sh 'sudo apt install redis-server -y'
                    sh 'sudo systemctl start redis-server'
                    sh 'sudo systemctl enable redis-server'
                    sh 'sudo service redis-server status'

                    sh 'sudo chmod -R 777 /var/lib/jenkins/workspace/Comify'
                    sh 'sudo chmod -R 777 /var/lib/jenkins/workspace/Comify/*'
                    sh 'sudo chown -R jenkins:www-data /var/lib/jenkins/workspace/Comify'
                    sh 'sudo chown -R jenkins:www-data /var/lib/jenkins/workspace/Comify/*'
                }
            }
        }

        stage('Install Celery') {
            steps {
                script {
                    sh 'sudo cp -rf DevOps/celery_comify.service /etc/systemd/system/'
                    sh 'sudo systemctl daemon-reload'

                    sh 'sudo systemctl stop celery_comify.service'
                    sh 'sudo systemctl start celery_comify.service'
                    sh 'echo "celery_comify.service has started."'

                    sh 'sudo systemctl enable celery_comify.service'
                    sh 'echo "celery_comify.service has been enabled."'

                    sh 'sudo systemctl status celery_comify.service'
                }
            }
        }

        stage('Configure Ngnix') {
            steps {
                script {
                    sh 'sudo cp -rf DevOps/comify.conf /etc/nginx/sites-available/comify'
                    try {
                        sh 'sudo rm /etc/nginx/sites-enabled/comify'
                    } catch (Exception e) {
                        echo "Nginx Config does'nt exist: ${e.message}"
                    }
                    sh 'sudo ln -s /etc/nginx/sites-available/comify /etc/nginx/sites-enabled'
                    sh 'sudo nginx -t'
                    sh 'sudo systemctl reload nginx'
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh 'sudo cp -rf DevOps/gunicorn.service /etc/systemd/system/'
                    sh 'sudo systemctl daemon-reload'

                    sh 'sudo systemctl start gunicorn'
                    sh 'echo "Gunicorn has started."'

                    sh 'sudo systemctl enable gunicorn'
                    sh 'echo "Gunicorn has been enabled."'

                    sh 'sudo systemctl status gunicorn'
                    sh 'sudo systemctl restart gunicorn'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment successful!'
        }
    }
}