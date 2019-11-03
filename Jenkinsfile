pipeline {
    agent none

    stages {
        stage('Deploy in Dev environment') {
            agent any
            steps {
                sh 'kubectl version'
                sh 'kubectl apply -f namespace.yaml'
                sh 'kubectl apply -f . --recursive'
            }
        }
        stage('propose changes for QA') {
            agent any
            environment {
                PR_NUMBER = "build-$BUILD_NUMBER"
                DEVELOPMENT_IMAGE = sh(script: 'kubectl apply --dry-run -f app1/app.yaml -o jsonpath=\'{.spec.template.spec.containers[?(@.name == "hello-pod")].image}\'', , returnStdout: true).trim()
            }
            steps {
                git branch: 'master', credentialsId: 'github', url: 'git@github.com:testing-field/gitops-qa-infrastructure.git'
                sh 'git config --global user.email "walter.dalmut@gmail.com"'
                sh 'git config --global user.name "Walter Dal Mut"'

                sh "git checkout -b feature/${PR_NUMBER}"

                sh 'kubectl patch -f app1/app.yaml -p \'{"spec":{"template":{"spec":{"containers":[{"name":"hello-pod","image":"'+ DEVELOPMENT_IMAGE +'"}]}}}}\' --local -o yaml | tee /tmp/app1.yaml'
                sh 'mv /tmp/app1.yaml app1/app.yaml'

                sh 'git add app1/app.yaml'

                sh 'git commit -m "[JENKINS-CI] - new application release"'

                sshagent(['github']) {
                    sh("""
                        #!/usr/bin/env bash
                        set +x
                        export GIT_SSH_COMMAND="ssh -oStrictHostKeyChecking=no"
                        git push origin feature/${PR_NUMBER}
                     """)
                }

                withCredentials([string(credentialsId: 'github_token', variable: 'GITHUB_TOKEN')]) {
                    sh 'hub pull-request -m "[JENKINS-CI] - Pull Request for image: "'+DEVELOPMENT_IMAGE
                }
            }
        }
    }
}
