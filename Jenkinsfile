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
     } 
  }
}
}
 