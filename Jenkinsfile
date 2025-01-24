pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID='05abcb4d-5de3-4807-98c5-244ff769cbc6'
        NETLIFY_AUTH_TOKEN=credentials('netlify-token')
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
                          #test -f build/index.html
                          npm test
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
            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
            reuseNode true
            }
        }
        steps {
          sh '''
            npm install serve
            node_modules/.bin/serve -s build &
            sleep 12
            npx playwright test --reporter=html
            '''
        }
        post {
          always {
              publishHTML([allowMissing:false, alwaysLinkToLastBuild:false, keepAll:false, reportDir: 'playwright-report1', reportFiles: 'index.html', reportName:'Playwright Local', reportTitles: '', useWrapperFileDirectly:true])
          }
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
                  npm install netlify-cli
                  node_modules/.bin/netlify --version
                  echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                  node_modules/.bin/netlify deploy --dir=build --prod
                '''
           }
        }
         stage('Prod E2E') {
          agent {
            docker {
              image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
              reuseNode true
            }
          }
          environment {
            CI_ENVIRONMENT_URL ='https://genuine-licorice-9e9083.netlify.app'
          }
          steps {
            sh '''
               npx playwright test --reporter=html
               '''
          }
          post {
            always {
              publishHTML([allowMissing:false, alwaysLinkToLastBuild:false, keepAll:false, reportDir: 'playwright-report2', reportFiles: 'index.html', reportName:'Playwright E2E', reportTitles: '', useWrapperFileDirectly:true])
            }
          }
        }
  }
}