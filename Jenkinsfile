pipeline {
    agent any

    stages {
        stage('Build') {
            agent{
                docker{
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

        stage('Tests'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                echo 'Test stage'
                sh '''
                #test -f build/index.html
                npm test
                '''
            }
            post {
                always{
                    junit testResults: 'test-result/junit.xml', allowEmptyResults: true
                }
            }
    }
        }

        stage('Deploy') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_module/.bin/netlify --version
                '''
            }
        }

    
   
}
