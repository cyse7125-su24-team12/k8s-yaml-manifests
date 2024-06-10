 pipeline{
    agent any 
    tools { 
        nodejs 'Node 20' 
    }
     stages {
        stage('Checkout') {
          steps {
            checkout([$class: 'GitSCM', 
                branches: [[name: '*/main']],
                extensions: [[$class: 'CleanCheckout']],
                userRemoteConfigs: [[url: "https://github.com/cyse7125-su24-team12/static-site.git", credentialsId: 'shyam-github-credentials']]
                ])
            }
        }
        stage('Shell test'){
            steps{
                sh '''
                pwd 
                node --version 
                '''                
            }
        }
        stage('Fetch Latest Commit Message') {
            steps {
                script {
                    // Fetch the latest commit message
                    env.LATEST_COMMIT_MESSAGE = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    echo "Latest commit message: ${env.LATEST_COMMIT_MESSAGE}"
                }
            }
        }
        stage('Setup commitlint') {
            steps {
                script {
                    // Install commitlint CLI and conventional config
                    sh 'npm install --save-dev @commitlint/cli @commitlint/config-conventional'
                    // Create a commitlint configuration file
                    writeFile file: 'commitlint.config.js', text: """
                        module.exports = {extends: ['@commitlint/config-conventional']};
                    """
                }
            }
        }
        stage('Check Latest Commit Message') {
            steps {
                script {
                    // Fetch the latest commit message
                    def latestCommitMessage = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
                    writeFile file: 'latest_commit_message.txt', text: latestCommitMessage

                    // Lint the latest commit message
                    sh '''
                        # Use npx to run commitlint on the latest commit message
                        npx commitlint < latest_commit_message.txt
                    '''
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
