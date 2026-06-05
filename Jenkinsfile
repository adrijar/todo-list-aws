pipeline {
    agent { label 'principal' }

    environment {
        STACK_NAME = 'todo-list-aws-production'
        REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/adrijar/todo-list-aws.git'
                sh 'curl -o samconfig.toml https://raw.githubusercontent.com/adrijar/todo-list-aws-config/production/samconfig.toml'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region ${REGION}
                    sam deploy --config-env production --no-confirm-changeset --no-fail-on-empty-changeset
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    BASE_URL=$(aws cloudformation describe-stacks \
                        --stack-name ${STACK_NAME} \
                        --region ${REGION} \
                        --query "Stacks[0].Outputs[?OutputKey=='BaseUrlApi'].OutputValue" \
                        --output text)
                    export BASE_URL
                    pytest test/integration/todoApiTest.py -v -k "listtodos or gettodo"
                '''
            }
        }
    }
}
