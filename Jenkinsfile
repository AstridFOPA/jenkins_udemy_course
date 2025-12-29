pipeline {
    agent any

    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    cd learn-jenkins-app-main
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la 
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            steps {
            sh '''
                cd learn-jenkins-app-main
                test -f build/index.html
                npm test
            '''
            }

        }
    }
}