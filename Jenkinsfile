pipeline {
    agent any

    environment {
        FLASK_APP = 'app.py'
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
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-venv python3-pip default-libmysqlclient-dev
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
                sshagent (credentials: ['ec2-key-pair']) {
                    sh '''
                        scp -o StrictHostKeyChecking=no -r * ec2-user@<EC2-INSTANCE-IP>:/home/ec2-user/simple-webapp-flask
                        ssh -o StrictHostKeyChecking=no ec2-user@<EC2-INSTANCE-IP> << EOF
                        sudo apt-get update
                        sudo apt-get install -y python3 python3-venv python3-pip default-libmysqlclient-dev
                        cd /home/ec2-user/simple-webapp-flask
                        python3 -m venv venv
                        source venv/bin/activate
                        pip install -r requirements.txt
                        FLASK_APP=app.py nohup flask run --host=0.0.0.0 --port=5000 &
                        exit
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

