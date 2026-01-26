pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = "a876d72c-ab41-4960-ae14-bf1d026105f0"
        NETLIFY_AUTH_TOKEN = credentials('jenkins-access')
    }

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
        stage ('Run test ') {
            parallel {
                stage('Unit Test') {
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
                    post {
                        always {
                            junit 'learn-jenkins-app-main/jest-results/junit.xml'
                        }
                    }
                }
                     

                stage('E2E Tests') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            cd learn-jenkins-app-main
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html

                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])        }
                    }
                }               
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    cd learn-jenkins-app-main
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --prod --dir=build --no-build
                '''
            }
        }
    }
}