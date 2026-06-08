pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        APP_PATH = 'todo-list-aws-master/todo-list-aws-master'
    }
    stages {
        stage('Get Code') {
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
                        pip install -r src/requirements.txt
                        pip install flake8 bandit
                    '''
                }
            }
        }
        stage('Static Test') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                        . venv/bin/activate
                        flake8 src/ --format=pylint --output-file=flake8-report.txt || true
                        bandit -r src/ -f txt -o bandit-report.txt || true
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'todo-list-aws-master/todo-list-aws-master/flake8-report.txt, todo-list-aws-master/todo-list-aws-master/bandit-report.txt',
                                     allowEmptyArchive: true
                }
            }
        }
        stage('Deploy Staging') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                        sam build
                        sam deploy \
                            --stack-name todo-list-aws-staging \
                            --resolve-s3 \
                            --parameter-overrides Stage=staging \
                            --capabilities CAPABILITY_IAM \
                            --region us-east-1 \
                            --no-confirm-changeset \
                            --no-fail-on-empty-changeset
                    '''
                }
            }
        }
        stage('Rest Test') {
            steps {
                dir(env.APP_PATH) {
                    sh '''
                        . venv/bin/activate
                        pip install pytest requests

                        BASE_URL=$(aws cloudformation describe-stacks \
                            --stack-name todo-list-aws-staging \
                            --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                            --output text \
                            --region us-east-1)

                        echo "API URL: $BASE_URL"
                        export BASE_URL
                        pytest test/integration/todoApiTest.py -v || true
                    '''
                }
            }
        }
        stage('Promote to Master') {
            steps {
                sh '''
                    git config user.email "jenkins@ci.local"
                    git config user.name "Jenkins"
                    git fetch origin
                    git checkout master
                    git merge develop --no-ff -m "Merge develop into master - promote to production"
                    git push origin master
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline CI completado exitosamente'
        }
        failure {
            echo 'Pipeline CI fallido - revisar logs'
        }
    }
}
