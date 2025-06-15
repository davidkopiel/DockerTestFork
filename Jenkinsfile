pipeline {
    agent any

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'TRIGGER_KEY', value: '$.action'],
                [key: 'JF_GIT_REPO', value: '$.repository.name'],
                [key: 'JF_GIT_PULL_REQUEST_ID', value: '$.number'],
                [key: 'JF_GIT_OWNER', value: '$.repository.owner.login']
            ],
            causeString: 'Triggered by GitHub PR event',
            token: 'MyJobToken',
            printContributedVariables: true,
            printPostContent: true,
            silentResponse: false
        )
    }

    environment {
        JF_GIT_PROVIDER = 'github'
        JF_URL = credentials('JF_URL')
        JF_ACCESS_TOKEN = credentials('JF_ACCESS_TOKEN')
        JF_GIT_TOKEN = credentials('JF_GIT_TOKEN')
    }

    stages {

        stage('Scan Pull Request in Docker') {
            agent {
                docker {
                    image 'python:3.11'
                    args '-u root'
                }
            }
            steps {
                echo "🔍 Start Scan Pull Request in Docker"

                sh '''
                    echo "📂 Current directory: $(pwd)"
                    echo "🧪 Checking if .git directory exists..."
                    ls -la

                    echo "📥 Installing dependencies..."
                    apt-get update && apt-get install -y curl git

                    echo "⬇️ Downloading Frogbot..."
                    curl -fL https://releases.jfrog.io/artifactory/frogbot/v2/2.9.2/getFrogbot.sh -o getFrogbot.sh

                    echo "🔐 Making script executable..."
                    chmod +x getFrogbot.sh

                    echo "🚀 Running Frogbot setup script..."
                    ./getFrogbot.sh

                    echo "📁 Repo state after download:"
                    ls -la

                    echo "📦 Running Frogbot scan..."
                    ./frogbot scan-pull-request || {
                        echo '❌ Frogbot failed. Dumping debug info:'
                        ls -la
                        cat frogbot.log || true
                        exit 1
                    }

                    echo "✅ Scan completed"
                '''
            }
        }
    }
}
