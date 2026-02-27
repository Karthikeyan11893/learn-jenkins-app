pipeline {
    agent any

    environment {
        NODE_VERSION = '18-alpine'
        PLAYWRIGHT_IMAGE = 'mcr.microsoft.com/playwright:v1.39.0-jammy'
    }

    stages {
        stage('Install') {
            agent {
                docker {
                    image "node:${NODE_VERSION}"
                    reuseNode true
                }
            }

            steps {
                sh 'npm ci'
            }
        }

        stage('Quality Check') {
            parallel {
                stage('Lint') {
                    agent {
                        docker {
                            image "node:${NODE_VERSION}"
                            reuseNode true
                        }
                    }

                    steps {
                        sh 'npm run lint'
                    }
                }

                stage('Unit Tests') {
                    docker {
                        image "node:${NODE_VERSION}"
                        reuseNode true
                    }

                    steps {
                        sh 'npm test -- --ci --reporters=default --reporters=jest-junit'
                    }
                }

                stage('Type Check') {
                    docker {
                        image "node:${NODE_VERSION}"
                        reuseNode true
                    }

                    steps {
                        sh 'npm run type-check'
                    }
                }
            }
        }

        stage('Build') {
            agent {
                docker {
                    image "node:${NODE_VERSION}"
                    reuseNode true
                }
            }
            steps {
                sh 'npm run build'
            }
        }

        stage('Security Scan') {
            agent {
                docker {
                    image "node:${NODE_VERSION}"
                    reuseNode true
                }
            }
            steps {
                sh 'npm audit --audit-level=high'
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image "${PLAYWRIGHT_IMAGE}"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm ci
                    npx serve -s build -l 3000 &
                    SERVER_PID=$!
                    sleep 10
                    npx playwright test --reporter=html
                    kill $SERVER_PID
                '''
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo "Deploying application..."
                // Add real deployment command here
                // Example:
                // sh './deploy.sh'
            }
        }
    }
    
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([
                allowMissing: true,
                alwaysLinkToLastBuild: false,
                keepAll: false,
                reportDir: 'playwright-report',
                reportFiles: 'index.html',
                reportName: 'Playwright HTML Report',
                reportTitles: '',
                useWrapperFileDirectly: true
            ])
        }
    }
}