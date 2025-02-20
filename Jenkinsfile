pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'deefd59d-33f9-4064-a1ca-ef783e9aa577'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    // Enable ANSI colors and skip the default checkout so we can fix workspace permissions first.
    options {
        ansiColor('xterm')
        skipDefaultCheckout true
    }

    stages {
        stage('Pre-Cleanup') {
            steps {
                // Fix any permission issues in the workspace before checkout.
                // Adjust the command if your Jenkins user is not allowed to run sudo without a password.
                sh '''
                    echo "hello from github"
                    echo "Cleaning workspace permissions..."
                    sudo chown -R jenkins:jenkins "$WORKSPACE" || echo "Pre-cleanup skipped (or insufficient permissions)."
                '''
            }
        }
        stage('Checkout') {
            steps {
                // Perform a fresh checkout.
                checkout scm
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    // Uncomment the next line if you want to force running as non-root:
                    // args '--user node'
                }
            }
            steps {
                sh '''
                    echo "Before permission fix inside container:"
                    ls -la
                    
                    echo "Fixing ownership of workspace inside container..."
                    chown -R node:node "$WORKSPACE" || true

                    echo "After permission fix:"
                    ls -la

                    echo "Setting HOME variable to workspace..."
                    export HOME=$WORKSPACE

                    echo "Checking Node.js and npm versions..."
                    node --version
                    npm --version

                    echo "Using a safe npm cache directory..."
                    export NPM_CONFIG_CACHE=$(pwd)/.npm

                    echo "Installing dependencies..."
                    npm ci --unsafe-perm

                    echo "Building project..."
                    npm run build

                    echo "Final workspace contents:"
                    ls -la
                '''
            }
        }
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    export HOME=$WORKSPACE
                    echo "Running tests..."
                    test -f build/index.html
                    npm test
                '''
            }
        }
        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                    args '--user root'
                }
            }
            steps {
                sh '''
                    export HOME=$WORKSPACE
                    echo "Installing Netlify CLI locally with --unsafe-perm..."
                    npm install netlify-cli --unsafe-perm

                    echo "Checking Netlify CLI version..."
                    $WORKSPACE/node_modules/.bin/netlify --version
                    $WORKSPACE/node_modules/.bin/netlify status
                    echo "from github"
                    echo "Deploying to Netlify. Site ID: $NETLIFY_SITE_ID"
                '''
            }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}
