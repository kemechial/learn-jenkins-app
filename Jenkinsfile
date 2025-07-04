pipeline {
    agent any

    environment {
       NETLIFY_SITE_ID = '099664cc-d5ba-465e-b335-5f5a2d6e31af'
       NETLIFY_AUTH_TOKEN = credentials('netlify-token')
       //REACT_APP_VERSION ='1.2.3'
       REACT_APP_VERSION ="1.2.$BUILD_ID"
    }

    stages {

        stage('Docker Build') {
           steps {
                sh '''
                    echo "Docker build stage"
                    docker build -t my-playwright .
                '''
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

        stage('Tests') {
            parallel {

                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo "Test stage"
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
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
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
                            publishHTML([
                                allowMissing: false,
                                alwaysLinkToLastBuild: false,
                                icon: '',
                                keepAll: false,
                                reportDir: 'playwright-report',
                                reportFiles: 'index.html',
                                reportName: 'Playwright Local',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
                }

            }
        }
        /*
         stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                 echo 'small change'
                   npm install netlify-cli node-jq
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --dir=build --no-build --json > output.json
                  '''
                   //node_modules/.bin/node-jq -r '.deploy_url' output.json
            script {

               def output = sh(
                    script: "node_modules/.bin/node-jq -r '.deploy_url' output.json",
                    returnStdout: true
                )

               env.STAGING_URL = output.trim()

            }
            
            
            
            
            }

            
            
        }
        */

            stage('Staging E2E') {
                    agent {
                        docker {
                            //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    
                    environment {
                        //CI_ENVIRONMENT_URL ='https://melodious-toffee-353ea7.netlify.app'
                        //CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
                        CI_ENVIRONMENT_URL = "TO BE SET"
                    }
                    
                    steps {
                        /*
                        sh '''
                            npm install netlify-cli node-jq
                            node_modules/.bin/netlify --version
                            echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                            node_modules/.bin/netlify status
                            node_modules/.bin/netlify deploy --dir=build --no-build --json > output.json
                            CI_ENVIRONMENT_URL=$(node_modules/.bin/node-jq -r '.deploy_url' output.json)
                            npx playwright test --reporter=html
                        '''
                        */
                        sh '''
                             node --version
                             netlify --version
                             echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                             netlify status
                             netlify deploy --dir=build --no-build --json > output.json
                             CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' output.json)
                             echo "Staging URL: $CI_ENVIRONMENT_URL"
                             npx playwright test --reporter=html
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
                                reportFiles: 'index.html',
                                reportName: 'Playwright E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
        }


        /*
        stage('Approval'){
            steps {
                timeout(15) {
                      input cancel: 'No...', message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                  }     
            }
          
        }


    

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                 echo 'small change'
                   npm install netlify-cli
                   node_modules/.bin/netlify --version
                   echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                   node_modules/.bin/netlify status
                   node_modules/.bin/netlify deploy --prod --dir=build --no-build 
                  
                '''
            }
        }

        */


        stage('Prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }

                    environment {
                        CI_ENVIRONMENT_URL ='https://melodious-toffee-353ea7.netlify.app'

                    }

                    steps {
                        sh '''
                             node --version
                             echo 'small change'
                             npm install netlify-cli
                             node_modules/.bin/netlify --version
                             echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                             node_modules/.bin/netlify status
                             node_modules/.bin/netlify deploy --prod --dir=build --no-build 
                             npx playwright test --reporter=html
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
                                reportFiles: 'index.html',
                                reportName: 'Playwright E2E',
                                reportTitles: '',
                                useWrapperFileDirectly: true
                            ])
                        }
                    }
        }

        /*

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Test stage"
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
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
        }

    
        */

    }
        /*
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
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])
        }
    }

    */


}