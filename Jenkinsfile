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
                    def targetBranch = "main"

                    withCredentials([usernamePassword(credentialsId: 'GITHUB_CRED', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                        sh "git fetch https://${GIT_USER}:${GIT_TOKEN}@github.com/thangngh/nestjs-jenkin-demo.git ${targetBranch}:${targetBranch}"

                        def mergeStatus = sh(
                          script: "git merge-tree \$(git merge-base HEAD ${targetBranch}) HEAD ${targetBranch}",
                          returnStdout: true
                        )

                        if (mergeStatus.contains('<<<<<<< ')) {
                          echo "⚠️ Conflicts detected in pull request!"
                          currentBuild.result = 'UNSTABLE'
                          error "Conflicts detected in pull request. Please resolve before merging."
                        } else {
                            echo "✅ No conflicts detected."
                        }
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
                            "ls -la && \\
                            cd nestjs-jenkin-demo && \\
                            git pull origin
                            "
                        """
                }
            }
        }
    }

    post {
        success {
            echo "Pull request validation completed successfully!"
            updateGitHubCommitStatus('SUCCESS', 'All checks passed')
        }
        unstable {
            echo "Pull request validation completed with warnings!"
            updateGitHubCommitStatus('UNSTABLE', 'Completed with warnings')
        }
        failure {
            echo "Pull request validation failed!"
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
            [$class: 'AnyBuildResult', message: message, state: state],
            [$class: 'BetterThanOrEqualBuildResult', result: 'FAILURE', state: 'FAILURE', message: build.description],
        ]]
    ])
}