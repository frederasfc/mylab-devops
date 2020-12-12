pipeline {
    agent any
    environment {
        registry = "frederasfc/devops"
        registryCredential = '123'
        dockerImage = ''
        SSH_PASSWORD = credentials('ssh-password')
    }
    stages {
        stage('Building our image') {
            steps{
                script {
                    def BRANCH = sh (script: "echo $GIT_BRANCH | cut -d'/' -f2", returnStdout: true ).trim()
                    echo "${BRANCH}"
                    dockerImage = docker.build registry + ":api_" + BRANCH + "_$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy our image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Cleaning up') {
            steps{
                script{
                    def BRANCH = sh (script: "echo $GIT_BRANCH | cut -d'/' -f2", returnStdout: true ).trim()
                    sh "docker rmi $registry:api_" + BRANCH + "_$BUILD_NUMBER"
                }
            }
        }
        stage('Changing deployment file') {
            steps{
                script{
                    sh "envsubst < deployment-up-api.yaml.tpl > deployment-up-api.yaml"
                }
            }
        }
        stage('Deploy on Fargate'){
            steps{
                script{
                    def BRANCH = sh (script: "echo $GIT_BRANCH | cut -d'/' -f2", returnStdout: true ).trim()
                    def remote = [:]
                    remote.name = 'master1'
                    remote.user = 'up'
                    remote.host = '192.168.100.10'
                    remote.password = "${SSH_PASSWORD}"
                    remote.allowAnyHosts = true
                    stage("SSH Steps Rocks!") {
                        sshPut remote: remote, from: 'k8-secret.sh', into: '.'
                        sshPut remote: remote, from: 'deployment-upl-api.yaml', into: '.'
                        sshCommand remote: remote, command: "chmod +x k8-secret.sh"
                        sshCommand remote: remote, command: "sudo ./k8-secret.sh"
                        sshCommand remote: remote, command: "sudo kubectl apply -f deployment-up-api.yaml  --kubeconfig=/root/.kube/config"
                        sshCommand remote: remote, command: 'sudo rm deployment-up-api.yaml'
                        sshCommand remote: remote, command: 'sudo rm k8-secret.sh'
                    }
                }
            }
        }
    }
}
