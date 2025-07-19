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

        stage('NPM Dependency Audit'){
            steps {
                sh '''
                    npm audit --audit-level=critical
                    echo $?
                '''
            }
        }
    }
}
