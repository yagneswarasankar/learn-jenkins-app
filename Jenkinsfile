pipeline {
    agent any
    stages {
        stage('Hello') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {'''
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