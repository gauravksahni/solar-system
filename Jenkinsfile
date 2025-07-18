pipeline {
    agent any 

    tools {
        nodejs 'nodejs244'
    }
    stages{
        stage('Check node version'){
            steps{
                sh '''
                    node -v
                    npm -v
                '''
            }
        }
    }
}
