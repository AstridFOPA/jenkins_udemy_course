pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        APP_NAME = "learnjenkinsapp"
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ECS_CLUSTER = "LearnJenkinsApp-Cluster-Prod"
        AWS_AWS_SERVICE_PROD = "LearnJenkinsApp-Service-Prod2"
        AWS_ECS_TD = "LearnJenkinsApp-TaskDefinition-Prod"
        AWS_DOCKER_REGISTRY = "864899864155.dkr.ecr.us-east-1.amazonaws.com"
    }

    stages {
        /* this is how to add a comment s
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
                            image 'my-playwright'
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

        stage('Docker_image build'){
            agent {
                docker {
                    image 'my-aws-cli'
                    args " -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint ''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    cd learn-jenkins-app-main 
                    docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                    docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
                }
            }
        }

        stage('deploy to AWS'){
            agent {
                docker {
                    image 'my-aws-cli'
                    args "--entrypoint ''"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version   
                    sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" learn-jenkins-app-main/aws/task-definition-prod.json
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://learn-jenkins-app-main/aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TD_REVISION
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_AWS_SERVICE_PROD --task-definition $AWS_ECS_TD:$LATEST_TD_REVISION
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_AWS_SERVICE_PROD
                '''
                }
            }
        }
    }
}
