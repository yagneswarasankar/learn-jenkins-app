pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'c54d7eba-43a2-4ada-9d2c-f614bc488079'
        NETLIFY_AUTH_TOKEN = credentials('jenkins-token')
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
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
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install netlify-cli node-jq
                            node_modules/.bin/netlify --version
                            echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build
                            node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
                        '''
                    }
                    script {
                    env.STAGING_URL = sh(script:"node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                    }
                }
                stage('staging E2E') {
                            agent {
                                docker {
                                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                    reuseNode true
                                }
                            }

                            environment {
                                CI_ENVIRONMENT_URL = $env.STAGING_URL
                            }

                            steps {
                                sh '''
                                    npx playwright test  --reporter=html
                                '''
                            }
            post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }


    }
        stage('Approval') {
//                             agent {
//                                 docker {
//                                     image 'node:18-alpine'
//                                     reuseNode true
//                                 }
//                             }
                            steps {
                            timeout(time: 15, unit: 'MINIUTES') {
                                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                            }

                            }
                        }
        stage('Deploy prod') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install netlify-cli
                            node_modules/.bin/netlify --version
                            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --prod
                        '''
                    }
                }

                stage('Prod E2E') {
                            agent {
                                docker {
                                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                                    reuseNode true
                                }
                            }

                            environment {
                                CI_ENVIRONMENT_URL = 'https://inspiring-khapse-95caee.netlify.app/'
                            }

                            steps {
                                sh '''
                                    npx playwright test  --reporter=html
                                '''
                            }
            post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Production E2E', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }


    }
}
}