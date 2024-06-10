pipeline{
    agent any 
    tools { 
        nodejs 'Node 20' 
    }
     stages {

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

                    # Loop through the commits and echo the commit messages
                    echo "$COMMITS" | jq -r '.[] | .commit.message' | while read COMMIT_MESSAGE; do
                        echo "Commit Message: $COMMIT_MESSAGE"
                    done
                    '''                
                }
            }
        }
        // stage('Setup Commitlint') {
        //     steps {
        //         sh """
        //             npm install -g @commitlint/cli @commitlint/config-conventional
        //             echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
        //         """
        //     }
        // }
        // stage('Fetch and Lint Commit Messages') {
        //     steps {
        //         script {
        //             // Fetching all commit messages from the current branch
        //             def commitMessages = sh(script: "git log --pretty=format:'%s'", returnStdout: true).trim()

        //             // Splitting commit messages into a list for individual processing
        //             def messageList = commitMessages.tokenize('\n')

        //             // Iterate over each commit message and perform linting
        //             messageList.each { message ->
        //                 writeFile file: 'temp_message.txt', text: message
        //                 // Perform commitlint check on each message
        //                 try {
        //                     sh 'commitlint < temp_message.txt'
        //                     echo "Commit message '${message}' passed linting."
        //                 } catch (Exception e) {
        //                     echo "Commit message '${message}' failed linting."
        //                     // Optionally, you can handle failures, e.g., by failing the build
        //                     // currentBuild.result = 'FAILURE'
        //                 }
        //             }
        //         }
        //     }
        // }
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
