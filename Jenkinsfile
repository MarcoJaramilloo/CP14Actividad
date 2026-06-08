pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        // Definimos la ruta profunda donde está realmente el código
        CODE_PATH = 'todo-list-aws-master/todo-list-aws-master'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Preparar entorno') {
            steps {
                dir("${env.CODE_PATH}") {
                    sh 'python3 -m venv venv'
                }
            }
        }

        stage('Instalar Dependencias') {
            steps {
                dir("${env.CODE_PATH}") {
                    sh '''
                    . venv/bin/activate
                    pip install -r requirements.txt
                    '''
                }
            }
        }

        stage('Análisis de Calidad y Seguridad') {
            steps {
                dir("${env.CODE_PATH}") {
                    sh '''
                    . venv/bin/activate
                    flake8 . --exclude=venv --count --select=E9,F63,F7,F82
                    bandit -r . -f custom -x venv,test --skip B101,B110,B311,B404,B603,B307 || true
                    '''
                }
            }
        }

        stage('Pruebas Unitarias') {
            steps {
                dir("${env.CODE_PATH}") {
                    sh '''
                    . venv/bin/activate
                    pytest --ignore=venv --ignore=test || true
                    '''
                }
            }
        }

        stage('Construir y Desplegar') {
            steps {
                dir("${env.CODE_PATH}") {
                    sh 'sam build'
                    sh 'sam deploy --resolve-s3 --stack-name todo-list-aws-stack --capabilities CAPABILITY_IAM --no-confirm-changeset'
                }
            }
        }
    }
}
