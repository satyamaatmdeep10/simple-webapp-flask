pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
        EC2_PRIVATE_KEY = credentials('ec2-key-pair')
        EC2_USER = 'ec2-user'
        EC2_HOST = '<EC2-INSTANCE-IP>'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/satyamaatmdeep10/simple-webapp-flask.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y python3 python3-venv python3-pip default-libmysqlclient-dev
                    python3 -m venv venv
                    source venv/bin/activate
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    source venv/bin/activate
                    pytest
                '''
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-key-pair']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
                        cd /path/to/your/project
                        git pull origin master
                        source venv/bin/activate
                        pip install -r requirements.txt
                        flask run --host=0.0.0.0 --port=5000 &
                        EOF
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'rm -rf venv'
        }
    }
}

