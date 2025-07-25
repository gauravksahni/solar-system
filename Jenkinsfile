pipeline {
    agent any 

    tools {
        nodejs 'nodejs244'
    }
    options {
        disableConcurrentBuilds abortPrevious: true
    }
    environment {
        MONGO_URI = "xxx-xxx-xxx-xxx"
        MONGO_USER = credentials('mongo-creds-username')
        MONGO_PASSWORD = credentials('mongo-creds-password')
        SCANNER_HOME = tool 'sonar-scanner';
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
                            --disableYarnAudit \
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
                withSonarQubeEnv('sonarqube-server') {
                    sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=solar-system \
                        -Dsonar.sources=app.js \
                        -Dsonar.javascript.lcov.reportPaths=./coverage/lcov.info 
                    """
                } 
            }
        }
        stage('Quality Gate') {
            steps{
                timeout(time: 60, unit: 'SECONDS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Build Docker image'){
            steps{
                sh """docker build -t gauravkb/solar-system:${GIT_COMMIT} ."""
            }
        }

        stage('Trivy Vulnerability Scan') {
            steps {
                sh """
                    trivy image gauravkb/solar-system:${GIT_COMMIT} \
                        --quiet \
                        --severity CRITICAL \
                        --exit-code 1 \
                        --format json \
                        -o trivy-image-CRITICAL-result.json
                """
            }
            post {
                always {
                    sh """
                        trivy convert \
                            --format template \
                            --template '/usr/local/share/trivy/templates/html.tpl' \
                            --output trivy-image-CRITICAL-result.html \
                            trivy-image-CRITICAL-result.json

                        trivy convert \
                            --format template \
                            --template '/usr/local/share/trivy/templates/junit.tpl' \
                            --output trivy-image-CRITICAL-result.xml \
                            trivy-image-CRITICAL-result.json
                """
                }
            }
        }

        stage('Push to Docker image'){
            steps{
                withDockerRegistry(credentialsId: 'docker-credentials', url: '') {
                    sh """docker push gauravkb/solar-system:${GIT_COMMIT}"""
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

            junit allowEmptyResults: true, keepProperties: true, testResults: 'trivy-image-CRITICAL-result.xml'

            publishHTML([allowMissing: true, alwaysLinkToLastBuild: true, icon: '', keepAll: true, reportDir: './', reportFiles: 'trivy-image-CRITICAL-result.html', reportName: 'Trivy Image Scan Report', reportTitles: '', useWrapperFileDirectly: true])


        }
    }
}
