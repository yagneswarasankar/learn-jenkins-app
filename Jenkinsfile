pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION='us-east-1'
        AWS_ECS_SERVICE_NAME='LearnJenkinsApp-Service-Prod'
        AWS_ECS_CLUSTER_NAME='LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_TASK_DEFINITION='LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
               stage('Build') {

            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker image'){
            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock  --entrypoint=''"
                }
            }
            steps{
                sh '''
                docker build  -t $APP_NAME:$REACT_APP_VERSION .
                '''
            }
        }
        stage('AWS') {

            agent {
                docker{
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                aws --version
                LATEST_TD_VERSION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                echo $LATEST_TD_VERSION
                aws ecs update-service --cluster $AWS_ECS_CLUSTER_NAME --service $AWS_ECS_SERVICE_NAME  --task-definition $AWS_ECS_TASK_DEFINITION:$LATEST_TD_VERSION
                aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER_NAME --services $AWS_ECS_SERVICE_NAME
                '''
                    }
               
            }
        }
    }
}