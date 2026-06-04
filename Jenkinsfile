pipeline {
    agent any

    environment {
        STACK_NAME = 'todo-list-aws-staging'
        REGION = 'us-east-1'
    }

    stages {

        stage('Get Code') {
            steps {
                git branch: 'develop', url: 'https://github.com/adrijar/todo-list-aws.git'
            }
        }

        stage('Static Test') {
            steps {
                sh '''
                    flake8 src/ --format=pylint > flake8-report.txt || true
                    bandit -r src/ -f txt -o bandit-report.txt || true
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: '*-report.txt', allowEmptyArchive: true
                }
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
                        --parameter-overrides Stage=staging
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
                    pytest test/integration/todoApiTest.py -v
                '''
            }
        }

       stage('Promote') {
    steps {
        withCredentials([usernamePassword(credentialsId: 'github-credentials', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
            sh '''
                git config user.email "adrian.simonriv0565@comunidadunir.net"
                git config user.name "adrijar"
                git fetch origin
                git checkout master
                git merge develop
                git checkout origin/master -- Jenkinsfile
                git commit -m "Restore CD Jenkinsfile after merge" || true
                git push https://${GIT_USER}:${GIT_TOKEN}@github.com/adrijar/todo-list-aws.git master --force
            '''
        }
    }
}
}
    }
}