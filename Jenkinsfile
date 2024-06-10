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

        stage('Fetch and Lint Commit Messages') {
            steps {
                script {
                    // Fetch all commit messages from the current branch
                    def commitMessages = sh(script: "git log --pretty=format:'%H %s'", returnStdout: true).trim()
                    def commits = commitMessages.tokenize('\n')

                    // Lint each commit message
                    commits.each { commit ->
                        def hash = commit.split()[0]
                        def message = commit.substring(commit.indexOf(' ') + 1)
                        writeFile file: 'temp_message.txt', text: message
                        echo "Linting commit: $hash"
                        sh 'commitlint < temp_message.txt' // Run commitlint on each message
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
