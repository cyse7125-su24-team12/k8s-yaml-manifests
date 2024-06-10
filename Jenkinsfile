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
                pwd 
                node --version 
                '''                
            }
        }
        stage('Setup Commitlint') {
            steps {
                sh """
                    npm install -g @commitlint/cli @commitlint/config-conventional
                    echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js
                """
            }
        }

        stage('Lint All Commits') {
            steps {
                script {
                    // Get all commit hashes since last successful build
                    def commitHashes = sh(script: "git rev-list HEAD ^last_successful_build --no-merges", returnStdout: true).trim()
                    // Split the output into an array of commit hashes
                    def commits = commitHashes.tokenize()

                    // Iterate over each commit and lint the commit message
                    commits.each { commit ->
                        def commitMessage = sh(script: "git log -1 --format=%B ${commit}", returnStdout: true).trim()
                        echo "Commit message: ${commitMessage}" // Echo each commit message
                        writeFile file: 'temp_commit_message.txt', text: commitMessage
                        sh 'commitlint < temp_commit_message.txt' // Lint each commit message
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Cleaning up...'
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
