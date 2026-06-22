pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '0ae649d9-d490-4670-8285-bffb138396b6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        // Test report configuration
        JEST_JUNIT_OUTPUT_DIR = 'test-result'
        JEST_JUNIT_OUTPUT_NAME = 'junit.xml'
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
                    echo "=== Build Stage ==="
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Test stage'
                sh '''
                    echo "=== Installing test reporters ==="
                    npm install --save-dev jest-junit || echo "jest-junit already installed"
                    
                    echo "=== Running tests ==="
                    npm test
                    
                    echo "=== Test results ==="
                    ls -la test-result/ || echo "No test-result folder found"
                '''
            }
            post {
                always {
                    // Publish test results for trending
                    junit testResults: 'test-result/*.xml', allowEmptyResults: true
                    
                    // Archive test results for historical data
                    archiveArtifacts artifacts: 'test-result/*.xml', allowEmptyArchive: true
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
            when {
                branch 'main'  // Only deploy from main branch
            }
            steps {
                sh '''
                    echo "=== Deploy Stage ==="
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod --site "$NETLIFY_SITE_ID" --auth "$NETLIFY_AUTH_TOKEN"
                    echo "=== Deployment complete! ==="
                '''
            }
        }
    }
    
    post {
        always {
            echo "=== Pipeline finished ==="
            
            // Always publish test results for trending
            junit testResults: 'test-result/*.xml', allowEmptyResults: true
            
            // Archive test results for historical trends
            archiveArtifacts artifacts: 'test-result/*.xml', allowEmptyArchive: true
            
            // Clean up workspace
            cleanWs()
        }
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            echo '❌ Pipeline failed!'
        }
    }
}