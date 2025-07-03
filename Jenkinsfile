pipeline {
   agent { 
       docker { 
           image 'mcr.microsoft.com/playwright:v1.53.2-noble'
           args '--user root'
       } 
   }
   stages {
       stage('SCM') {
           steps {
                git branch: 'main',
                url: 'https://github.com/cabalug/pw-poc'
           }
       }
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
                 steps {
                    sh '''
                        pnpm test:qa
                    '''
                 }
              }
               stage('Prod') {
                 steps {
                    sh '''
                        pnpm test:prod
                    '''
                 }
              }
           }
       }
   }
   post {
       always {
           publishHTML (target : [
             allowMissing: true,
             alwaysLinkToLastBuild: true,
             keepAll: true,
             reportDir: 'reports/playwright-report-qa',
             reportFiles: 'index.html',
             reportName: 'Playwright Report QA',
             reportTitles: 'Report QA'])
            
            publishHTML (target : [
             allowMissing: true,
             alwaysLinkToLastBuild: true,
             keepAll: true,
             reportDir: 'reports/playwright-report-prod',
             reportFiles: 'index.html',
             reportName: 'Playwright Report PROD',
             reportTitles: 'Report PROD'])
       }
   }
}