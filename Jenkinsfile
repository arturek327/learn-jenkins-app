pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID='05abcb4d-5de3-4807-98c5-244ff769cbc6'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
        REACT_APP_VERSION="1.0.$BUILD_ID"

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
        
         stage ('AWS') {
          agent{
            docker {
              image  'amazon/aws-cli'
              args "--entrypoint=''"
              reuseNode true
            }
          }
          environment {
            AWS_S3_BUCKET='dlubek-learn-jenkins'
          }
          steps {
            withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
              sh '''
               aws --version
               echo "Hello S3!" > index.html
               aws s3 sync build s3://$AWS_S3_BUCKET
             '''
            }
          }
        }

        stage ('Tests'){
          parallel {
                   stage ('Unit tests'){
                    agent {
                            docker {
                              image 'node:18-alpine'
                              reuseNode true
                            }
                          }
                      steps {
                        sh '''
                          echo abrakadabra1
                          test -f build/index.html
                          npm test
                          echo abrakadabra2
                          '''
                      }
                      post {
                          always {
                            junit 'jest-results/junit.xml'
                          }
                      }
          }
        stage ('E2E') {
        agent {
          docker {
            image 'my-playwright'
            reuseNode true
            }
        }
        steps {
          sh '''
            serve -s build &
            sleep 12
            npx playwright test --reporter=html
            '''
        }
        post {
          always {
              publishHTML([allowMissing:false, alwaysLinkToLastBuild:false, keepAll:false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName:'Playwright Local', reportTitles: '', useWrapperFileDirectly:true])
          }
        }
     }
          }
        }
        stage('Deploying E2E') {
          agent {
            docker {
              image 'my-playwright'
              reuseNode true
            }
          }
          environment {
            CI_ENVIRONMENT_URL='STAGING_URL_TO_BE_SET'
          }
          steps {
            sh '''
               netlify --version
               echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
               netlify deploy --dir=build --json > deploy-output.json
               CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' deploy-output.json)
               npx playwright test --reporter=html
               '''
          }
          post {
            always {
              publishHTML([allowMissing:false, alwaysLinkToLastBuild:false, keepAll:false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName:'Staging E2E', reportTitles: '', useWrapperFileDirectly:true])
            }
          }
        }
         stage('Deploy prod') {
          agent {
            docker {
              image 'my-playwright'
              reuseNode true
            }
          }
          environment {
            CI_ENVIRONMENT_URL ='https://genuine-licorice-9e9083.netlify.app'
          }
          steps {
            sh '''
                netlify --version
                echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                netlify deploy --dir=build --prod
               npx playwright test --reporter=html
               '''
          }
          post {
            always {
              publishHTML([allowMissing:false, alwaysLinkToLastBuild:false, keepAll:false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName:'Prod E2E', reportTitles: '', useWrapperFileDirectly:true])
            }
          }
        }
  }
}