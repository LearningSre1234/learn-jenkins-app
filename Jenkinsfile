pipeline{

    agent any

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



    post{
        success{
            archiveArtifacts artifacts: '**'
        }

        always{
            echo '----------------------------------------------------------------------------'
            junit 'jest-results/junit.xml'
            echo '----------------------------------------------------------------------------'
        }
    }
}