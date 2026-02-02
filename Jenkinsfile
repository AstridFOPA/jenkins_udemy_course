pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION = "us-east-1"
        AWS_ECS_CLUSTER = "LearnJenkinsApp-Cluster-Prod"
        AWS_AWS_SERVICE_PROD = "LearnJenkinsApp-TaskDefinition-Prod"
        AWS_ECS_TD = "LearnJenkinsApp-TaskDefinition"
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

        stage('Docker_image build'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args " -u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint ''"
                    reuseNode true
                }
            steps {
                sh ''' 
                    amazon-linux-extras install docker
                    service docker start
                    docker build -t my-jenkinsapp .
                '''
            }
        }
    }    

        stage('deploy to AWS'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args " -u root --entrypoint ''"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version   
                    yum install -y jq
                    echo  "HELLO S3!!" > index.html
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
