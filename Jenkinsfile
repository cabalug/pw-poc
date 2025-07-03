pipeline {
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }

    parameters {
        choice(name: 'ENV_FILTER', choices: ['all', 'qa', 'st', 'prod'], description: 'Run on specific platform')
        choice(name: 'FEATURE_FILTER', choices: ['all', 'Auth', 'Events', 'Supply'], description: 'Run specific feature')
    }
    
    agent { 
        docker { 
            image 'mcr.microsoft.com/playwright:v1.53.2-noble'
            args '--user root'
        } 
    }
    
    stages {
        stage('Build') {
            steps {
                sh '''
                    corepack enable pnpm
                    pnpm install
               '''
            }
        }
       
        stage('Test') {
            parallel {
                stage('QA') {
                    when { anyOf {
                        expression { params.ENV_FILTER == 'all' }
                        expression { params.ENV_FILTER == 'qa' }
                    }}
                    
                    steps {
                        sh '''
                            pnpm test:qa -g ${FEATURE_FILTER}
                        '''
                    }
                }
                
                stage('ST') {
                    when { anyOf {
                        expression { params.ENV_FILTER == 'all' }
                        expression { params.ENV_FILTER == 'st' }
                    }}
                    
                    steps {
                        sh '''
                            pnpm test:st -g ${FEATURE_FILTER}
                        '''
                    }
                }
                
                stage('PROD') {
                    when { anyOf {
                        expression { params.ENV_FILTER == 'all' }
                        expression { params.ENV_FILTER == 'prod' }
                    }}
                    steps {
                        sh '''
                            pnpm test:prod -g ${FEATURE_FILTER}
                        '''
                    }
                }
            }
        }
    }
   
    post {
        always {
            script {
                def environments = ['qa', 'st', 'prod']

                environments.each { envKey ->
                    if (params.ENV_FILTER == 'all' || params.ENV_FILTER == envKey) {
                        publishHTML(target: [
                            allowMissing: true,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: "reports/playwright-report-${envKey}",
                            reportFiles: 'index.html',
                            reportName: "Playwright Report ${envKey.toUpperCase()}",
                            reportTitles: "Playwright Report ${envKey.toUpperCase()}"
                        ])
                    }
                }
            }
        }
    }
}