pipeline {
    agent any 

    tools {
        nodejs 'nodejs244'
    }
    stages{
        stage('Install dependencies'){
            steps{
                sh 'npm install --no-audit'
            }
        }
    }
}
