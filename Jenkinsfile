pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c54d7eba-43a2-4ada-9d2c-f614bc488079'
        NETLIFY_AUTH_TOKEN = credentials('jenkins-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
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

         stage('AWS') {

            environment{
                AWS_S3_BUCKET_NAME = "jenkins-learning-160220251748"
            }
            agent {
                docker{
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    euseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                aws s3 sync build s3://$AWS_S3_BUCKET_NAME/
                '''
                    }
               
            }
        }


        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            #test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

        stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwrite'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy staging') {
                            agent {
                                docker {
                                    image 'my-playwrite'
                                    reuseNode true
                                }
                            }

                            environment {
                                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
                            }

                            steps {
                                sh '''
                                    netlify --version
                                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                                    netlify status
                                    netlify deploy --dir=build --json > deploy-output.json
                                    CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                                    npx playwright test --reporter=html
                                '''
                            }
            post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }


               }

    stage('Deploy prod') {
                agent {
                   docker {
                     image 'my-playwrite'
                     reuseNode true
                           }
                   }
                environment {
                     CI_ENVIRONMENT_URL = 'https://inspiring-khapse-95caee.netlify.app/'
                   }
                steps {
                      sh '''
                         node --version
                         netlify --version
                         echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                         netlify status
                         netlify deploy --dir=build --prod
                         npx playwright test  --reporter=html
                         '''
                       }
                      post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
    stage("execute java"){
         agent{
                docker {
                    image 'oraclelinux:8'
                    }
                }
                steps{
                sh 'java docker_test.java'
                }
            }

        }
}