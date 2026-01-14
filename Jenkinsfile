pipeline {
    agent any

    stages {
        /* this is how to add a comment 
        in Jenkinsfile */
        stage('Build') {
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
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test

                '''
            }
        }
    }
    post {
        always {
            junit 'learn-jenkins-app-main/jest-results/junit.xml'
        }
    }
}