pipeline {
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 60, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    triggers {
        githubPush()
    }

    stages {

        stage ('Start') {
            steps {
                script {
                    updateGitHubCommitStatus('PENDING', 'Jenkins is validating the pull request...')
                }
            }
        }

        stage ('Validation PR') {
            steps {
                script {
                    echo 'Validating PR...'
                }
            }
        }

        stage ('Checkout') {
            steps {
                script {
                    checkout scm
                }
            }
        }

        stage ('Check Conflict') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {

                    }
                }
            }
        }

        stage ('Build') {
            when {
                // expression { return env.BRANCH_NAME == 'main' }
                branch 'main'
            }
            steps {
                script {
                    echo 'Building...'
                }
            }
        }

        stage ('Test') {
            steps {
                script {
                    echo 'Running tests...'
                }
            }
        }

        stage ('Deploy production') {
            when {
                // expression { return env.BRANCH_NAME == 'main' }
                branch 'main'
            }
            steps {
                script {
                    echo 'Deploying to production...'
                }
            }
        }

        stage ('SSH agent') {
            when {
                // expression { return env.BRANCH_NAME == 'main' }
                branch 'main'
            }
            steps {
                sshagent(credentials: ['df464007-da47-414c-907d-7c46364d9075']) {
                    sh  """
                            ssh -o StrictHostKeyChecking=no root@192.168.20.250 \\
                            "ls -la"
                        """
                }
            }
        }
    }

    post {
        success {
            updateGitHubCommitStatus('SUCCESS', 'All checks passed')
        }

        unstable {
            updateGitHubCommitStatus('UNSTABLE', 'Completed with warnings')
        }

        failure {
            updateGitHubCommitStatus('FAILURE', 'Build failed')
        }

        always {
            cleanWs(deleteDirs: true, disableDeferredWipeout: true)
        }
    }
}

void updateGitHubCommitStatus(String state, String message) {
    step([
        $class: 'GitHubCommitStatusSetter',
        contextSource: [$class: 'ManuallyEnteredCommitContextSource', context: 'Jenkins PR Validation'],
        errorHandlers: [[$class: 'ChangingBuildStatusErrorHandler', result: 'UNSTABLE']],
        statusResultSource: [$class: 'ConditionalStatusResultSource', results: [
            [$class: 'AnyBuildResult', message: message, state: state]
        ]]
    ])
}