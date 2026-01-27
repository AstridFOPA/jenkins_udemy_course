pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = "a876d72c-ab41-4960-ae14-bf1d026105f0"
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
    }

    stages {
        /* this is how to add a comment 
        in Jenkinsfile */

        stage('AWS'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args " --entrypoint ''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    echo  "HELLO S3!!" > index.html
                    aws s3 cp index.html s3://learn-jenkins-20260126/index.html
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
                            image 'my-playwright'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            cd learn-jenkins-app-main
                            serve -s build &
                            sleep 10
                            npx playwright test --reporter=html 

                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'playwright local', reportTitles: '', useWrapperFileDirectly: true])        }
                    }
                }               
            }
        }

        stage('stagging E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL= "STAGING_URL_TO_BE SET"
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    netlify --version
                    echo "deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --no-build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(jq -r \'.deploy_url\' deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])        }
            }
        }

        stage('prod Deploy-E2E TESTS') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "https://incandescent-toffee-235889.netlify.app"
            }

            steps {
                sh '''
                    cd learn-jenkins-app-main
                    netlify --version
                    echo "deploying to production. ASTRID Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod --no-build
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'learn-jenkins-app-main/playwright-report', reportFiles: 'index.html', reportName: 'playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])        }
            }
        }
    }
}