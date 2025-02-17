pipeline {
    agent any

    options {
        ansiColor('xterm')
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
                    echo "Before permission fix:"
                    ls -la
                    
                    echo "Fixing ownership of workspace..."
                    chown -R node:node /var/lib/jenkins/workspace || true

                    echo "After permission fix:"
                    ls -la
                    
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
                    test -f build/index.html
                    npm test
                '''
            }
        }

        stage ('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Fixing npm permissions..."
                    mkdir -p $WORKSPACE/.npm-global
                    mkdir -p $WORKSPACE/.npm-cache
                    chmod -R 777 $WORKSPACE/.npm-global $WORKSPACE/.npm-cache

                    echo "Setting npm global install path..."
                    export NPM_CONFIG_PREFIX=$WORKSPACE/.npm-global
                    export PATH=$WORKSPACE/.npm-global/bin:$PATH
                    export NPM_CONFIG_CACHE=$WORKSPACE/.npm-cache

                    echo "Installing Netlify CLI locally..."
                    npm install netlify-cli

                    echo "Checking Netlify CLI version..."
                    $WORKSPACE/.npm-global/bin/netlify --version
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
