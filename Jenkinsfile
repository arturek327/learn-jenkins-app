pipeline {
    agent any
    environmnet {
       NETLIFY_SITE_ID='05abcb4d-5de3-4807-98c5-244ff769cbc6'
       NETLIFY_AUTH_TOKEN=credential('netlify-toke')
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
  
        stage ('Run Tests'){
          parallel {
                   stage ('Test'){
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
                  node_modules/.bin/netlify status 
                '''
            }
        }
          }
        }
 
    }
}