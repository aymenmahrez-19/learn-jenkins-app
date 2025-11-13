pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '13361cfa-f5e4-4de5-99d5-a684c6a903d0'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {

        stage('Build') {
            agent { docker { image 'node:18-alpine' reuseNode true } }
            steps {
                sh 'npm ci && npm run build'
            }
        }

        stage('Unit Tests') {
            agent { docker { image 'node:18-alpine' reuseNode true } }
            steps {
                sh '''
                    npm install --no-save babel-jest @babel/preset-react jest-junit
                    mkdir -p jest-results
                    npx jest --ci --reporters=default --reporters=jest-junit --outputFile=jest-results/junit.xml
                '''
            }
            post {
                always {
                    junit 'jest-results/junit.xml'
                }
            }
        }

        stage('E2E Tests') {
            agent { docker { image 'mcr.microsoft.com/playwright:v1.39.0-jammy' reuseNode true } }
            steps {
                sh '''
                    npx playwright install --with-deps
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([
                        reportDir: 'playwright-report',
                        reportFiles: 'index.html',
                        reportName: 'Playwright HTML Report'
                    ])
                }
            }
        }

        stage('Deploy') {
            when { expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') } }
            agent { docker { image 'node:18-alpine' reuseNode true } }
            steps {
                sh '''
                    npm install netlify-cli
                    npx netlify deploy --dir=build --prod --auth=$NETLIFY_AUTH_TOKEN --site=$NETLIFY_SITE_ID
                '''
            }
        }
    }
}
