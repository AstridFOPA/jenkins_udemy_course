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
        stage('test') {
            steps {
                sh '''
                ls -la ./build | grep index.html
                cd learn-jenkins-app-main
                npm run test
                '''
            }
        }
    }
}

