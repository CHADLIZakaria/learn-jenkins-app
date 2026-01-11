pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f44a8090-b912-43b7-8281-b49e58403c6e'
        NETLIFY_AUTH_TOKEN = credentials('netlify_token')
        REACT_APP_VERSION = "1.2.$BUILD_ID"
        AWS_DEFAULT_REGION='us-east-1'
    }

    stages {

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json
                    '''
                }
            }
        }

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

        /*        
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
                            image 'my-playwright'
                            reuseNode true
                        }               
                    }
                    steps {
                        sh '''
                            serve -s build &
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                script {
                    sh '''
                        netlify --version
                        echo "Deploy to staging Site Id: $NETLIFY_SITE_ID"
                        netlify status
                        netlify deploy --dir=build --json > deploy-output.json
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
                    image 'my-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL='https://steady-pavlova-ffcfe7.netlify.app'
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to production Site Id: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --prod
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
        */
    }
}