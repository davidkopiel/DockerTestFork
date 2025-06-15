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
        stage('Debug Info') {
            steps {
                echo "🧩 GIT REPO: ${JF_GIT_REPO}"
                echo "🧩 PR ID: ${JF_GIT_PULL_REQUEST_ID}"
                echo "🧩 OWNER: ${JF_GIT_OWNER}"
                echo "🧩 TRIGGER: ${TRIGGER_KEY}"
            }
        }

        stage('Scan Pull Request in Docker') {
            agent {
                docker {
                    image 'python:3.11'
                    args '-u root'
                }
            }
            steps {
                sh '''
                    set -e
                    apt-get update && apt-get install -y curl git

                    echo "📥 Cloning PR branch from GitHub..."
                    git clone https://github.com/${JF_GIT_OWNER}/${JF_GIT_REPO}.git repo
                    cd repo

                    echo "📌 Checking out pull request branch..."
                    git fetch origin pull/${JF_GIT_PULL_REQUEST_ID}/head:pr-branch
                    git checkout pr-branch

                    echo "🚀 Downloading Frogbot..."
                    curl -fL https://releases.jfrog.io/artifactory/frogbot/v2/2.9.2/getFrogbot.sh -o getFrogbot.sh
                    chmod +x getFrogbot.sh
                    ./getFrogbot.sh

                    echo "🔍 Running scan-pull-request..."
                    ./frogbot scan-pull-request
                '''
            }
        }
    }
}
