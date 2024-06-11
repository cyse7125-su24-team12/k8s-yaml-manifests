pipeline {
    agent any
    tools {
        nodejs 'Node 20'
    }
    environment {
        CURRENT_VERSION = currentVersion()
        NEXT_VERSION = nextVersion()
    }
    stages {
        stage('Setup Commitlint') {
            steps {
                sh """
                # Check if commitlint is already installed and install if not
                if ! npm list -g @commitlint/cli | grep -q '@commitlint/cli'; then
                    npm install -g @commitlint/cli
                fi

                if ! npm list -g @commitlint/config-conventional | grep -q '@commitlint/config-conventional'; then
                    npm install -g @commitlint/config-conventional
                fi

                # Ensure the commitlint config file is present
                if [ ! -f commitlint.config.js ]; then
                    echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
                fi
                """
            }
        }
        stage('Lint commit messages') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'git-credentials-id', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                    sh '''
                    node --version
                    echo " source branch: $CHANGE_BRANCH"
                    echo " target branch: $CHANGE_TARGET"
                    echo " url: $CHANGE_URL"

                    # Extract the owner and repository name from the CHANGE_URL
                    OWNER=$(echo "$CHANGE_URL" | sed 's|https://github.com/\\([^/]*\\)/\\([^/]*\\)/pull/.*|\\1|')
                    REPO=$(echo "$CHANGE_URL" | sed 's|https://github.com/\\([^/]*\\)/\\([^/]*\\)/pull/.*|\\2|')

                    # Extract the pull request number from the CHANGE_URL
                    PR_NUMBER=$(echo "$CHANGE_URL" | sed 's|.*/pull/\\([0-9]*\\).*|\\1|')

                    echo "Owner: $OWNER"
                    echo "Repository: $REPO"
                    echo "Pull Request Number: $PR_NUMBER"

                    # GitHub API endpoint to get commits from a specific pull request
                    API_URL="https://api.github.com/repos/$OWNER/$REPO/pulls/$PR_NUMBER/commits"

                    # Make an authenticated API request to get the commits
                    COMMITS=$(curl -s -H "Authorization: token $GIT_PASSWORD" "$API_URL")

                    # Parse and lint each commit message
                    echo "$COMMITS" | jq -r '.[] | .commit.message' | while read COMMIT_MESSAGE; do
                        echo "$COMMIT_MESSAGE" | npx commitlint
                        if [ $? -ne 0 ]; then
                            echo "Commit message linting failed."
                            exit 1
                        fi
                    done
                    '''
                }
            }
        }
        stage('Get next version of the application ') {
            steps {
                sh '''
                echo "Current version: $CURRENT_VERSION"
                echo "Next version: $NEXT_VERSION"
                '''
            }
        }
        stage('Checkout') {
                    steps {
                        checkout([$class: 'GitSCM',
                            branches: [[name: '*/main']],
                            extensions: [[$class: 'CleanCheckout']],
                            userRemoteConfigs: [[url: 'https://github.com/cyse7125-su24-team12/k8s-yaml-manifests.git', credentialsId: 'git-credentials-id']]
                        ])
                    }
        }
        stage('Validate YAML file') {
            steps {
                script {
                    sh 'sudo apt-get update && sudo apt-get install -y yamllint'
                    sh 'yamllint .'
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning up...'
        }
        success {
            echo 'Build Succeeded for the commit message linting and yaml file validation.'
        }
        failure {
            echo 'Build Failed'
        // Actions to handle failure, e.g., notify, mark build as unstable
        }
    }
}
