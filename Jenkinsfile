pipeline {
    agent any

    stages {
        /*
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
        */
        stage('Test') {
             agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
             }
             steps {
                 sh '''
                    # echo "Test Stage"
                    test -f build/index.html
                    npm test
                 '''

            }
            
        }

        stage('E2E') {
             agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                    reuseNode true
                }
             }
             steps {
                 sh '''
                    npm install serve
                    npx playwright install
                    mkdir -p test-results playwright-report
                    chown -R $(id -u):$(id -g) jest-results playwright-report
                    node_modules/.bin/serve -s build &
                    SERVE_PID=$!
                    sleep 10
                    npx playwright test --reporter=html
                    kill $SERVE_PID
                 '''

            }
            
        }
    }
    post {
         always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwrght HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
