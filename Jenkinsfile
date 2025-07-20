pipeline {
    agent any 

    tools {
        nodejs 'nodejs244'
    }
    environment {
        MONGO_URI = "xxx-xxx-xxx-xxx"
        MONGO_USER = credentials('mongo-creds-username')
        MONGO_PASSWORD = credentials('mongo-creds-password')
        SONAR_TOKEN = credentials('sonar-solar-system-token')
        SCANNER_HOME = tool 'sonar-scanner';
        SONAR_SCANNER_OPTS = '-Xmx1024m'
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
                    }
                } 
            }
        }
        // stage('Unit Testing'){
        //     options {
        //         timestamps()
        //         retry(2)
        //     }
        //     steps {
        //         sh 'npm test' 
        //     }
        // }  
        stage('Code Coverage'){
            steps{
                catchError(buildResult: 'SUCCESS', message: 'OOPS. will be fixed in next release', stageResult: 'UNSTABLE') {
                    sh 'npm run coverage'
                }
            }
        }

        stage('SAST Sonarqube') {
            steps {
                withEnv([
                    "SCANNER_HOME=${tool 'sonar-scanner'}",
                    "SONAR_SCANNER_OPTS=-Xmx2048m"
                ]) {
                    sh """
                        \$SCANNER_HOME/bin/sonar-scanner \\
                        -Dsonar.projectKey=solar-system \\
                        -Dsonar.sources=. \\
                        -Dsonar.host.url=http://192.168.59.121:9000 \\
                        -Dsonar.token=${SONAR_TOKEN}
                    """
                }
            }
        }

    }
    post {
        always {
            junit allowEmptyResults: true, keepProperties: true, testResults: 'dependency-check-junit.xml'

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'dependency-check-report.html', reportName: 'Dependency Check HTML Report', reportTitles: '', useWrapperFileDirectly: true])

            junit allowEmptyResults: true, keepProperties: true, testResults: 'test-report.xml'

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: 'coverage/lcov-report', reportFiles: 'index.html', reportName: 'Code Coverage HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
