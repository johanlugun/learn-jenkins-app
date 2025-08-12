pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '1570a216-ba27-458a-95f2-00e0c57236b6'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

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
            parallel{
                stage('Unit Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            test -f build/index.html
                            npm test
                        '''                
                    }
                    post{
                        always{
                            junit skipPublishingChecks: true,
                            testResults: 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            npx playwright install chromium
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''                
                    }                    
                    post{
                        always{
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Local - Playwright HTML Report',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }
            }
        }
        stage('Deploy Staging'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = "STAGING_URL"
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json)                    
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''                
            }                    
            post{
                always{
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Stage - Playwright HTML Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
        stage('Deploy Prod'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                    reuseNode true
                }
            }
            environment{
                CI_ENVIRONMENT_URL = 'https://majestic-truffle-4fb732.netlify.app'
            }
            steps{
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright install chromium
                    npx playwright test --reporter=html
                '''                
            }                    
            post{
                always{
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        icon: '',
                        keepAll: false,
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Prod - Playwright HTML Report',
                        reportTitles: '',
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }    
}
