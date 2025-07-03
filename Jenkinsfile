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
                    
                    script {
                        def command = 'pnpm test:qa'
                        if (params.FEATURE_FILTER != 'all') {
                            command += " -g '${params.FEATURE_FILTER}'"
                        }
                        sh command
                    }
                }
                
                stage('ST') {
                    when { anyOf {
                        expression { params.ENV_FILTER == 'all' }
                        expression { params.ENV_FILTER == 'st' }
                    }}
                    
                    script {
                        def command = 'pnpm test:st'
                        if (params.FEATURE_FILTER != 'all') {
                            command += " -g '${params.FEATURE_FILTER}'"
                        }
                        sh command
                    }
                }
                
                stage('PROD') {
                    when { anyOf {
                        expression { params.ENV_FILTER == 'all' }
                        expression { params.ENV_FILTER == 'prod' }
                    }}
                   
                    script {
                        def command = 'pnpm test:prod'
                        if (params.FEATURE_FILTER != 'all') {
                            command += " -g '${params.FEATURE_FILTER}'"
                        }
                        sh command
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