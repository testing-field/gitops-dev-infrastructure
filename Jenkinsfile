pipeline {
    agent none

    stages {
        stage('Setup') {
            agent any
            steps {
                withCredentials([file(credentialsId: 'gpg_key', variable: 'FILE'), string(credentialsId: 'gpg_key_pass', variable: 'GPG_PASS')]) {
                    sh 'echo $GPG_PASS | gpg --batch --import $FILE'
                    sh 'echo $GPG_PASS > key.txt'
                    sh 'touch dummy.txt'
                    sh 'gpg --batch --yes --passphrase-file key.txt --pinentry-mode=loopback -s dummy.txt'
                    sh 'sops -i --decrypt db/config.yaml'
                }
            }
        }
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
                DEVELOPMENT_IMAGE_APP1 = sh(script: 'kubectl apply --dry-run -f app1/app.yaml -o jsonpath=\'{.spec.template.spec.containers[?(@.name == "hello-pod")].image}\'', , returnStdout: true).trim()
                DEVELOPMENT_IMAGE_APP2 = sh(script: 'kubectl apply --dry-run -f app2/app.yaml -o jsonpath=\'{.spec.template.spec.containers[?(@.name == "hello-pod")].image}\'', , returnStdout: true).trim()
            }
            steps {
                git branch: 'master', credentialsId: 'github', url: 'git@github.com:testing-field/gitops-qa-infrastructure.git'
                sh 'git config --global user.email "walter.dalmut@gmail.com"'
                sh 'git config --global user.name "Walter Dal Mut"'

                sh "git checkout -b feature/${PR_NUMBER}"

                sh 'kubectl patch -f app1/app.yaml -p \'{"spec":{"template":{"spec":{"containers":[{"name":"hello-pod","image":"'+ DEVELOPMENT_IMAGE_APP1 +'"}]}}}}\' --local -o yaml | tee /tmp/app1.yaml'
                sh 'mv /tmp/app1.yaml app1/app.yaml'
                sh 'git add app1/app.yaml'

                sh 'kubectl patch -f app2/app.yaml -p \'{"spec":{"template":{"spec":{"containers":[{"name":"hello-pod","image":"'+ DEVELOPMENT_IMAGE_APP2 +'"}]}}}}\' --local -o yaml | tee /tmp/app2.yaml'
                sh 'mv /tmp/app2.yaml app2/app.yaml'
                sh 'git add app2/app.yaml'

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
