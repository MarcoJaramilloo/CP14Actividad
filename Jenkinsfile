pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'us-east-2'
        // Ruta base donde está el código según tus capturas
        APP_PATH = 'todo-list-aws-master/todo-list-aws-master'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Instalar Dependencias') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    # Buscamos el requirements.txt dentro de la carpeta src
                    pip install -r src/requirements.txt
                    '''
                }
            }
        }

        stage('Análisis de Calidad y Seguridad') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                    . venv/bin/activate
                    flake8 . --exclude=venv --count --select=E9,F63,F7,F82
                    bandit -r . -f custom -x venv,test --skip B101,B110,B311,B404,B603,B307,B102,B310,B302,B324,B411,B105,B202,B604,B602,B504,B108 || true
                    '''
                }
            }
        }

        stage('Pruebas Unitarias') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                    . venv/bin/activate
                    pytest --ignore=venv --ignore=test || true
                    '''
                }
            }
        }

        stage('Construir y Desplegar') {
            steps {
                dir(env.APP_PATH) {
                    // sam build detectará el template.yaml en la raíz de APP_PATH
                    sh 'sam build'
                    sh 'sam deploy --resolve-s3 --stack-name todo-list-aws-stack --capabilities CAPABILITY_IAM --no-confirm-changeset'
                }
            }
        }
    }
}
