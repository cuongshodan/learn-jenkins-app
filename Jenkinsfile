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
            steps {
                sh 'echo "Testing..."'
            }
        }
    }
}
