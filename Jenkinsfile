pipeline {
    agent any

    environment {
        PYTHON = 'python3'
        VENV_DIR = "${WORKSPACE}/venv"
        DEPLOY_DIR = "${WORKSPACE}/python-app-deploy"
    }

    stages {
        stage('Setup Environment') {
            steps {
                echo 'Checking and installing Python if needed...'
                sh '''
                if ! command -v python3 &> /dev/null; then
                    echo "Python3 not found! Please install it before running the pipeline."
                    exit 1
                fi
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Creating virtual environment and installing dependencies...'
                sh '''
                ${PYTHON} -m venv ${VENV_DIR}
                source ${VENV_DIR}/bin/activate
                pip install --upgrade pip
                if [ -f requirements.txt ]; then
                    pip install -r requirements.txt
                fi
                deactivate
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh '''
                source ${VENV_DIR}/bin/activate
                ${PYTHON} -m unittest discover -s .
                deactivate
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                sh '''
                mkdir -p ${DEPLOY_DIR}
                cp ${WORKSPACE}/app.py ${DEPLOY_DIR}/
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Running application...'
                sh '''
                source ${VENV_DIR}/bin/activate
                nohup ${PYTHON} ${DEPLOY_DIR}/app.py > ${DEPLOY_DIR}/app.log 2>&1 &
                echo $! > ${DEPLOY_DIR}/app.pid
                deactivate
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Testing application...'
                sh '''
                source ${VENV_DIR}/bin/activate
                ${PYTHON} ${WORKSPACE}/test_app.py
                deactivate
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for more details.'
        }
    }
}

