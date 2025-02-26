pipeline {
    agent any
    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        APP_NAME='myjenkinsapp'
        AWS_DEFAULT_REGION="us-east-1"
        AWS_ECS_CLUSTER ='LearnJenkinsApp-Cluster-Preprod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearningJenkinsApp-TaskDefinition-Preprod'
        AWS_DOCKER_REGISTRY = '886436949623.dkr.ecr.us-east-1.amazonaws.com'
    }
    stages {
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
                  node --version
                  npm --version
                  npm ci
                  npm run build
                  ls -la
                '''
            }
        }
        stage ('Build Docker image') {
          agent{
            docker {
              image  'my-aws-cli'
              args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
              reuseNode true
            }
          }
          steps {
              withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                   docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                   aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                   docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                  '''
              }
          }
        }
         stage ('deploy to AWS') {
          agent{
            docker {
              image  'my-aws-cli'
              args "--entrypoint=''"
              reuseNode true
            }
          }
          steps {
            withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
              sh '''
               aws --version
               LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json|jq '.taskDefinition.revision')
               aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
               aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
             '''
            }
          }
       }
   }
}
