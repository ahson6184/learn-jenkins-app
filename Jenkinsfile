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
        stage('Tests') {
            parallel{

                stage('Unit Test') {
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
        
                    post {
                            always {
                                junit 'jest-results/junit.xml'   
                            }
                        }
                }    

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                            reuseNode true
                            args '-u 1000:1000'
                        }
                    }
                    steps {

                        cleanWs()  // Cleans everything except the pipeline script itself

                        sh '''
                            npm install serve

                            # Recreate directories with correct permissions
                            mkdir -p test-results playwright-report
                            
                            
                            # Start server
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            sleep 10
                            
                            npx playwright test --reporter=html
                            kill $SERVE_PID || true   
                        '''
                    }
                    post {
                        always {
                            publishHTML([
                                allowMissing: false, 
                                alwaysLinkToLastBuild: false, 
                                icon: '', 
                                keepAll: false, 
                                reportDir: 'playwright-report', 
                                reportFiles: 'index.html', reportName: 
                                'Playwrght HTML Report', 
                                reportTitles: '', 
                                useWrapperFileDirectly: true
                                ])
                        }
                    }
                    
                }
            }
        }
    }
    
}
