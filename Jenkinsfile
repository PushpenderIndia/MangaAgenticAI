pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                script {
                    git branch: 'main', credentialsId: 'be931eed-297a-4c57-9706-565d76161ee0', url: 'https://github.com/PushpenderIndia/MangaAgenticAI'
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

                    sh 'sudo chmod -R 777 /var/lib/jenkins/workspace/MangaAgenticAI'
                    sh 'sudo chmod -R 777 /var/lib/jenkins/workspace/MangaAgenticAI/*'
                    sh 'sudo chown -R jenkins:www-data /var/lib/jenkins/workspace/MangaAgenticAI'
                    sh 'sudo chown -R jenkins:www-data /var/lib/jenkins/workspace/MangaAgenticAI/*'
                }
            }
        }

        stage('Install Celery') {
            steps {
                script {
                    sh 'sudo cp -rf DevOps/celery_manga.service /etc/systemd/system/'
                    sh 'sudo systemctl daemon-reload'

                    sh 'sudo systemctl stop celery_manga.service'
                    sh 'sudo systemctl start celery_manga.service'
                    sh 'echo "celery_manga.service has started."'

                    sh 'sudo systemctl enable celery_manga.service'
                    sh 'echo "celery_manga.service has been enabled."'

                    sh 'sudo systemctl status celery_manga.service'
                }
            }
        }

        stage('Configure Ngnix') {
            steps {
                script {
                    sh 'sudo cp -rf DevOps/manga.conf /etc/nginx/sites-available/manga'
                    try {
                        sh 'sudo rm /etc/nginx/sites-enabled/manga'
                    } catch (Exception e) {
                        echo "Nginx Config does'nt exist: ${e.message}"
                    }
                    sh 'sudo ln -s /etc/nginx/sites-available/manga /etc/nginx/sites-enabled'
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