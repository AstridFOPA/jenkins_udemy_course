pipeline {
    agent any
    environment {
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION = "us-east-1"
    }

    stages {
        /* this is how to add a comment 
        in Jenkinsfile */
        stage('deploy to AWS'){
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args " --entrypoint ''"
                    reuseNode true
                }
            }

            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    echo  "HELLO S3!!" > index.html
                    aws ecs register-task-definition --cli-input-json file://learn-jenkins-app-main/aws/task-definition-prod.json
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


    }
}