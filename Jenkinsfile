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
                    args "--entrypoint=''"
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                ls -lrt
                chmod 777 aws/task-definition-prod.json
                cat ./aws/task-definition-prod.json
                aws --version
                aws ecs update-service --cluster LearningJenkins-Cluster-Prod --service LearningJenkinsApp-Service-Prod  --task-definition LearnJenkinsApp-TaskDefinition-Prod:1
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