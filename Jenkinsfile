pipeline{

    agent any

    environment{
        NETLIFY_SITE_ID = '76986d73-25c4-45a6-89a6-f076073afa3e'
        NETLIFY_AUTH_TOKEN = credentials('Netlify [Jenkins-APP]')
    }

    stages{
        // stage (cleanUp){
        //     steps{
        //         cleanWs()
        //     }
        // }

        stage('Build'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
                echo 'In Building Block'
                sh '''
                    ls -la
                    npm --version
                    node --version
                    npm ci
                    npm run build
                '''
            }
            
        }

        stage('Test'){
            parallel {
                stage('Jest Test'){
            
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps{
                        echo 'In Testing Stage'
                        sh '''
                            npm test
                        '''
                    }

                    post{
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E'){
                    agent{
                        docker{
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    

                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }

                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Stg-Deploy'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > Stg-Deploy.json
                '''

                script{
                    env.Stg_Var = sh(script: 'node_modules/.bin/node-jq -r ".deploy_url" Stg-Deploy.json', returnStdout: true)
                }
            }
        }


        stage('Stg -E2E'){
            agent{
                docker{
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment{
                    CI_ENVIRONMENT_URL = "${env.Stg_Var}"
                }
            
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'Stg-Test.html', reportName: 'PlayWright STG HTML Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                    // cleanWs()
                }
            }
        }

        stage('Approval'){
            steps{
                input message: '\'Ready aaa Bro\'', ok: 'Ready !'
            }
        }

        stage('Prod Deploy'){
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }

            steps{
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }


        stage('Prod -E2E'){
            agent{
                    docker{
                        image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                        reuseNode true
                    }
                }

            environment{
                    CI_ENVIRONMENT_URL = 'https://jocular-cat-1427f7.netlify.app/'
                }
            
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post{
                always{
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'Prod-Test.html', reportName: 'PlayWright HTML Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                    // cleanWs()
                }
            }
        }
        
    }
}