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
        stage('Dependency Scanning') {
            parallel{
                stage('NPM Dependency Audit'){
                    steps {
                        sh '''
                            npm audit --audit-level=critical
                            echo $?
                        '''
                    }
                }
                stage('OWASP Dependency Check'){
                    steps{
                        dependencyCheck additionalArguments: '''  
                            --scan \'./\' 
                            --out \'./\'  
                            --format \'ALL\' 
                            --prettyPrint''', odcInstallation: 'OWASP'

                        dependencyCheckPublisher failedTotalCritical: 1, pattern: 'dependency-check-report.xml', stopBuild: true

                        junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

                        publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                    }
                } 
            }
        }
        stage('Unit Testing'){
            steps {
                sh 'npm test'
            }
        }  
    }
}
