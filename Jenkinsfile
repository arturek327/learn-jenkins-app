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
            echo 1aaa
            npm install serve
            echo 3aaaa
            node_modules/.bin/serve -s build &
#            sleep 10
#            npx playwright test --reporter=html
            '''
        }
     } 
  }
}
}
}
 