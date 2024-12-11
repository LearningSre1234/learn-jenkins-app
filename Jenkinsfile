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
    }



    post{
        success{
            archiveArtifacts artifacts: '**'
        }

        always{
            junit 'test-results/junit.xml'
        }
    }
}