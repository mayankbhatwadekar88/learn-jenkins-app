pipeline {
        agent any
        environment {
            NETLIFY_SITE_ID='cc11f269-4166-40f2-956f-6fe836c660c7'
  	    NETLIFY_AUTH_TOKEN=credentials('netify-token')
        }
        stages {
            stage('Build') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
            steps {
                sh '''
                    hostname
                    ls -la
                    npm --version
                    node --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
            }
            stage('Tests')
            {
                parallel {
                    stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                    test -f build/index.html
                    npm test
                '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                    }
                stage('E2E test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=line
                '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
                }
            }
            stage('Deploy') {
                agent {
                    docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
		    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
		    node_modules/.bin/netlify status
	            node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
            }
	  stage('Prod E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.58.2-noble'
                            reuseNode true
                        }
                    }
                    environment {
                    CI_ENVIRONMENT_URL = 'jocular-florentine-5bd948.netlify.app'
        }
                    steps {
                        sh '''
                    npx playwright test --reporter=line
                '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
        }
}

