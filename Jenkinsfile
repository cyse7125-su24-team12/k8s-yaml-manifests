 pipeline{
    agent any 
    tools { 
        nodejs 'Node 20' 
    }
     stages {
        // stage('Checkout') {
        //   steps {
        //     checkout([$class: 'GitSCM', 
        //         // branches: [[name: '*/main']],
        //         extensions: [[$class: 'CleanCheckout']],
        //         userRemoteConfigs: [[url: "https://github.com/cyse7125-su24-team12/static-site.git", credentialsId: 'shyam-github-credentials']]
        //         ])
        //     }
        // }
        stage('Shell test'){
            steps{
                sh '''
                node --version 
                echo " source branch: $CHANGE_BRANCH"
                echo " target branch: $CHANGE_TARGET"
                echo " url: $CHANGE_URL"
                git log --pretty=format:'%s'
                # Find the fork point (common ancestor)
                fork_point=$(git merge-base $CHANGE_BRANCH $CHANGE_TARGET)

                # Show commits from the fork point up to the head of the feature branch
                git log --oneline $fork_point..$CHANGE_BRANCH

                '''                
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
