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
                        sh '''
                            # Clean up any previous test files
                            rm -rf test-results playwright-report || true
                            
                            # Recreate directories with correct permissions
                            mkdir -p test-results playwright-report
                            
                            # Install serve (if not in package.json)
                            npm install serve
                            
                            # Start server
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            
                            # Wait for server to start
                            sleep 10
                            
                            # Run tests with explicit output directories
                            npx playwright test \
                                --output=test-results \
                                --reporter=html,line \
                                --reporter-html-output=playwright-report
                            
                            # Verify results
                            ls -la playwright-report
                            
                            # Clean up server
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
