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
        stage('test') {
            steps {
            sh '''
                cd learn-jenkins-app-main
                ls -la ./build | grep index.html
                npm test
            '''
            }

        }
    }
}