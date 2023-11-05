pipeline {
    agent {
        docker {
            image 'python3.9-slim' // Указываем образ Python 3
        }
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Set up Python') {
            steps {
                script {
                    sh 'echo "*** Checking the Python version..."'
                    sh 'python3 --version'
                }
            }
        }

        stage('Set up environment') {
            steps {
                script {
                    sh 'echo "*** Creating a Python virtual environment"'
                    sh 'python3 -m venv ~/venv'
                    sh 'echo "*** Installing Selenium and Chrome for BDD"'
                    sh 'sudo apt-get update'
                    sh 'sudo DEBIAN_FRONTEND=noninteractive apt-get install -y sqlite3 ca-certificates chromium-driver python3-selenium'
                    sh 'echo "*** Installing Python dependencies..."'
                    sh 'source ~/venv/bin/activate && python3 -m pip install --upgrade pip wheel'
                    sh 'source ~/venv/bin/activate && pip install -r requirements.txt'
                }
            }
        }

        stage('Start PostgreSQL container') {
            steps {
                script {
                    sh 'docker run -d --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=postgres -v postgres:/var/lib/postgresql/data postgres:alpine'
                    sh 'docker ps'
                }
            }
        }

        stage('Analysing the code with pylint') {
            steps {
                script {
                    sh 'source ~/venv/bin/activate && python3 -m pylint $(git ls-files \'*.py\') || true'
                }
            }
        }

        stage('Run unit tests') {
            steps {
                script {
                    sh 'source ~/venv/bin/activate && nosetests --with-coverage'
                }
            }
        }

        stage('Run Flask app') {
            steps {
                script {
                    sh 'source ~/venv/bin/activate'
                    sh 'export FLASK_APP=service/__init__.py'
                    sh 'flask run --host=0.0.0.0 &'
                    sh 'sleep 5'
                }
            }
        }

        stage('Run BDD tests') {
            steps {
                script {
                    sh 'source ~/venv/bin/activate && python3 -m behave'
                }
            }
        }
    }
}
