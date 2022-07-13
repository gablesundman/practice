pipeline {
    agent any

    options {
        // See: https://plugins.jenkins.io/lockable-resources/
        lock resource: 'gherkin-sync'
    }

    environment {
        TOKEN = credentials('gherkin-sync-token')
        EMAIL = credentials('gherkin-sync-email')
        SCRIPT_REPO_URL = credentials('gherkin-sync-repo-url')
        EXT = 'feature'
        PARENT = 'Gherkin - Test'
        DIRECTORY = '../test'
    }

    stages {
        stage('Sync') {
            agent {
                docker {
                    image 'python:3.10-buster'
                }
            }
            steps {
                script {
                  // Clone the script repository.
                  sh "git clone ${SCRIPT_REPO_URL} gherkin-sync"
                  // Remove the script repository's .git folder.
                  sh 'rm -rf ./gherkin-sync/.git'
                }
                dir('gherkin-sync') {
                    withEnv(["HOME=${env.WORKSPACE}"]) {
                        // Get Pipenv ready.
                        sh 'pip install pipenv --user'
                        sh 'python -m pipenv install'
                        // Sync Gherkin feature files with Confluence via script.
                        sh """
                        python -m pipenv run ./sync.py --conf-token ${env.TOKEN} --email ${env.EMAIL} \\
                        --extension ${env.EXT} --parent ${env.PARENT} --directory ${env.DIRECTORY} \\
                        --build-url ${env.BUILD_URL}
                        """
                    }
                }
            }
        }
    }

    // Clean up the mess we've made.
    post {
        always {
            cleanWs()
        }
    }
}
