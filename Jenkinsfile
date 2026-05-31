pipeline {
    agent any

    environment {
        STACK_NAME = 'todo-list-aws-production'
        REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'master', url: 'https://github.com/adrijar/todo-list-aws.git'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sam build
                    sam validate --region ${REGION}
                    sam deploy \
                        --stack-name ${STACK_NAME} \
                        --region ${REGION} \
                        --capabilities CAPABILITY_IAM \
                        --no-confirm-changeset \
                        --no-fail-on-empty-changeset \
                        --s3-bucket todo-list-aws-bucket-787991874982 \
                        --parameter-overrides Stage=production
                '''
            }
        }

        stage('Rest Test') {
            steps {
                sh '''
                    BASE_URL=$(aws cloudformation describe-stacks \
                        --stack-name ${STACK_NAME} \
                        --region ${REGION} \
                        --query "Stacks[0].Outputs[?OutputKey==\`BaseUrlApi\`].OutputValue" \
                        --output text)
                    export BASE_URL
                    pytest test/integration/todoApiTest.py -v -k "listtodos or gettodo"
                '''
            }
        }
    }
}
