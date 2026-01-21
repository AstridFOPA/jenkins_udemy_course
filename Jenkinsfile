pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = "a876d72c-ab41-4960-ae14-bf1d026105f0"
        NETLIFY_AUTH_TOKEN = credentials('netlify-creadential')
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
                            npx playwright test --reporter=line

                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'playwright local', reportTitles: '', useWrapperFileDirectly: true])        }
                    }
                }               
            }
        }

        stage('deploy staging') {
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
                    npm install node-jq
                    node_modules/.bin/netlify --version
                    echo "deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --no-build --json > deploy-output.json
                '''
                script {
                    env.STAGING_URL = sh(script: "cd learn-jenkins-app-main && node_modules/.bin/node-jq -r \'.deploy_url\' deploy-output.json", returnStdout: true)
                }
            }
        }
        stage('stagging E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    npx playwright test --reporter=line
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])        }
            }
        }
        stage ( 'validation') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                   input message: 'Ready to deploy ?', ok: 'Yes, I\'m sure I want to deploy.' 
                }
            }
        }
        stage('prod Deploy') {
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
                    echo "deploying to production. ASTRID Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod --no-build
                '''
            }
        }

        stage('Prod E2E Tests') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "https://incandescent-toffee-235889.netlify.app"
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    npx playwright test --reporter=line
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])        }
            }
        }
    }
}