pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']],
                    extensions: [[$class: 'CleanCheckout']],
                    userRemoteConfigs: [[url: "https://github.com/cyse7125-su24-team12/k8s-yaml-manifests.git", credentialsId: 'git-credentials-id']]
                ])
            }
        }
        stage('Validate YAML file') {
            steps {
                script {
                    sh 'yamllint .'
                }
            }
        }
    }

    post {
        success {
            echo 'YAML file validation is success'
        }
        failure {
            echo 'YAML file validation is failure'
        }
    }
}
