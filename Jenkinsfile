pipeline {
    agent any 

    
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
