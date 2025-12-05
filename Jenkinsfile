pipeline {
    agent any

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
                    set -eux
                    echo "=== Build stage ==="
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            set -eux
                            echo "=== Unit tests stage ==="
                            echo "PWD: $(pwd)"
                            echo "Contents before tests:"
                            ls -la

                            # install deps for this container
                            npm ci

                            # run tests (your package.json already has --testResultsProcessor="jest-junit")
                            npm test

                            echo "---- listing test-results (if any) ----"
                            ls -R test-results || echo "no test-results directory"
                        '''
                    }
                    post {
                        always {
                            // Look for any junit XML under test-results (covers default jest-junit paths)
                            junit 'test-results/**/*.xml'
                        }
                    }
                }

            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    set -eux
                    echo "=== E2E stage ==="
                    npm install serve
                    node_modules/.bin/serve -s build &
                    sleep 10
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML(
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'playwright-report', 
                        reportFiles: 'index.html', 
                        reportName: 'Staging E2E', 
                        reportTitles: '', 
                        useWrapperFileDirectly: true
                    )
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
                    set -eux
                    echo "=== Deploy stage ==="
                    npm install netlify-cli -g
                    netlify --version
                    # add your netlify deploy command here later
                '''
            }
        }

    }
}
