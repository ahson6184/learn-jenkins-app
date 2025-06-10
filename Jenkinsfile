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
                            # Install only serve (Playwright is already installed in the image)
                            npm install serve
                            
                            # Start server (no permission issues since we're already 1000:1000)
                            node_modules/.bin/serve -s build &
                            SERVE_PID=$!
                            
                            # Wait for server to start
                            sleep 10
                            
                            # Run tests - no need for special permissions
                            npx playwright test --reporter=html
                            
                            # Clean up
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
