pipeline {
    agent any

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
        DEPLOY_DIR = "${WORKSPACE}/python-app-deploy"
    }

    stages {
        stage('Setup') {
            steps {
                echo 'Checking Python installation...'
                sh 'python3 --version || { echo "Python3 is not installed!"; exit 1; }'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                sh '''
                python3 -m venv $VENV_DIR
                source $VENV_DIR/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        
        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh '''
                source $VENV_DIR/bin/activate
                python3 -m unittest discover -s .
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                mkdir -p $DEPLOY_DIR
                cp app.py $DEPLOY_DIR/
                '''
            }
        }
        
        stage('Run Application') {
            steps {
                echo 'Stopping old application if running...'
                sh '''
                if [ -f $DEPLOY_DIR/app.pid ]; then
                    kill $(cat $DEPLOY_DIR/app.pid) || true
                    rm -f $DEPLOY_DIR/app.pid
                fi
                '''
                echo 'Starting application...'
                sh '''
                source $VENV_DIR/bin/activate
                nohup python3 $DEPLOY_DIR/app.py > $DEPLOY_DIR/app.log 2>&1 &
                echo $! > $DEPLOY_DIR/app.pid
                '''
            }
        }
        
        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                source $VENV_DIR/bin/activate
                python3 test_app.py || true
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs for more details.'
        }
    }
}

