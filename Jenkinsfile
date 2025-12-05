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
                            echo "Contents before install:"
                            ls -la || true

                            # install dependencies (ensures jest and any reporter deps are present)
                            npm ci

                            # If jest-junit isn't declared in package.json, install it as a fallback.
                            if ! (cat package.json 2>/dev/null | grep -q "jest-junit"); then
                              echo "jest-junit not found in package.json — installing locally"
                              npm install --no-audit --no-fund --save-dev jest-junit
                            else
                              echo "jest-junit already present (or package.json missing)"
                            fi

                            echo "Running tests (with jest-junit reporter fallback)..."
                            # Run tests. Passing reporter args ensures junit xml generation even if config missing.
                            # The `|| true` keeps the script continuing so we can debug files; remove it if you want strict failure behavior.
                            npm test -- --reporters=default --reporters=jest-junit --outputDirectory=jest-results --outputName=junit.xml || true

                            echo "---- after test: ls jest-results ----"
                            ls -la jest-results || true

                            if [ -f jest-results/junit.xml ]; then
                              echo "JUNIT FILE FOUND"
                              echo "---- junit.xml head ----"
                              sed -n '1,120p' jest-results/junit.xml || true
                            else
                              echo "NO JUNIT FILE: printing workspace for debug"
                              ls -la
                            fi
                        '''
                    }
                    post {
                        always {
                            // Archive JUnit-format test results so Jenkins shows test results
                            junit 'jest-results/junit.xml'
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
                    # You can add netlify deploy command here if you have NETLIFY_AUTH_TOKEN and site ID configured
                '''
            }
        }

    }
}
