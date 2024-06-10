pipeline{
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
                    npm install -g @commitlint/cli @commitlint/config-conventional
                    echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
                """
            }
        }
        stage('Shell test'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'github-credential-shyam', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
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
        stage('Get next version of the application '){
            steps{

                sh '''
                echo "Current version: $CURRENT_VERSION"
                echo "Next version: $NEXT_VERSION"
                '''
            }
        }
    }
    post {
        always {
            echo 'Cleaning up...'
            sh 'rm -f temp_message.txt'
            // Perform any necessary cleanup tasks
        }
        success {
            echo 'Commit message follows conventional commit standards.'
        }
        failure {
            echo 'Commit message does not follow conventional commit standards.'
            // Actions to handle failure, e.g., notify, mark build as unstable
        }
    }
}
