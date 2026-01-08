pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f44a8090-b912-43b7-8281-b49e58403c6e'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
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
        
        stage('Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true         
                        }
                    }
                    
                    steps {
                        sh '''
                            test -f build/index.html
                            echo $?
                            npm test
                        '''
                    }
                }
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                            reuseNode true
                        }               
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        
        stage('Deploy staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }
            steps {
                script {
                    sh '''
                        npm install netlify-cli@latest
                        node_modules/.bin/netlify --version
                        echo "Deploy to staging Site Id: $NETLIFY_SITE_ID"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    '''

                    env.CI_ENVIRONMENT_URL = sh(
                        script: "node -e \"console.log(require('./deploy-output.json').deploy_url)\"",
                        returnStdout: true
                    ).trim()

                    echo "CI_ENVIRONMENT_URL = ${env.CI_ENVIRONMENT_URL}"

                    sh '''
                        npx playwright test --reporter=html
                    '''
                }
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        icon: '', 
                        keepAll: false, 
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Playwright Staging Report', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL='https://steady-pavlova-ffcfe7.netlify.app'
            }
            steps {
                sh '''
                    npm install netlify-cli@latest
                    node_modules/.bin/netlify --version
                    echo "Deploy to production Site Id: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                    publishHTML([
                        allowMissing: false, 
                        alwaysLinkToLastBuild: false, 
                        icon: '', 
                        keepAll: false, 
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Playwright Prod Report', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    ])
                }
            }
        }
    }
}