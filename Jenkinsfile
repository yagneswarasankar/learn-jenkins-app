pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION='us-east-1'
    }

    stages {

        stage('AWS') {

            agent {
                docker{
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                yum install -jq -y
                aws --version
                LATEST_TD_VERSION = $(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                echo $LATEST_TD_VERSION
                aws ecs update-service --cluster LearningJenkins-Cluster-Prod --service LearningJenkinsApp-Service-Prod  --task-definition LearnJenkinsApp-TaskDefinition-Prod:$LATEST_TD_VERSION
                '''
                    }
               
            }
        }

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
    }
}