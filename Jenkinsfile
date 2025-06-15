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
        stage('Verify Trigger') {
            steps {
                script {
                    if (env.TRIGGER_KEY != 'synchronize' && env.TRIGGER_KEY != 'opened') {
                        error("Aborting: Not a relevant pull request event: ${env.TRIGGER_KEY}")
                    }
                }
            }
        }

        stage('Download Frogbot') {
            steps {
                sh '''
                    curl -fL https://releases.jfrog.io/artifactory/frogbot/v2/2.9.2/getFrogbot.sh -o getFrogbot.sh
                    chmod +x getFrogbot.sh
                    ./getFrogbot.sh
                '''
            }
        }

        stage('Scan Pull Request') {
            steps {
                sh '''
                    pip install pipenv
                    ./frogbot scan-pull-request
                    '''
            }
        }
    }
}
