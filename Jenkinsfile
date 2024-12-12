pipeline{

    agent any

    environment{
        NETLIFY_SITE_ID = '76986d73-25c4-45a6-89a6-f076073afa3e'
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
                }
            }
        }



        stage('Deploy'){
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
                '''
            }
        }

        
    }



    post{
        success{
            archiveArtifacts artifacts: '**'
        }

        always{
            echo '----------------------------------------------------------------------------'
            junit 'jest-results/junit.xml'
            echo '----------------------------------------------------------------------------'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PlayWright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}