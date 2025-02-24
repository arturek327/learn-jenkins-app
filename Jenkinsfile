pipeline {
    agent any
    environment {
        REACT_APP_VERSION="1.0.$BUILD_ID"
        AWS_DEFAULT_REGION="us-east-1"
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
        
         stage ('deploy to AWS') {
          agent{
            docker {
              image  'amazon/aws-cli'
              args "-u root --entrypoint=''"
              reuseNode true
            }
          }
          steps {
            withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
              sh '''
               aws --version
               yum install jq -y
               LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json|jq '.taskDefinition.revision')
               echo $LATEST_TD_REVISION
               aws ecs update-service --cluster LearnJenkinsApp-Cluster-Preprod --service LearnJenkinsApp-Service-Prod --task-definition LearningJenkinsApp-TaskDefinition-Preprod:$LATEST_TD_REVISION
               aws ecs wait services-stable --cluster LearnJenkinsApp-Cluster-Preprod --service LearnJenkinsApp-Service-Prod
             '''
            }
          }
       }
   }
}
