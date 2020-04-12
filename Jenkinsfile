pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
        REGISTRY = "markwu100/nodeapp"
        REGISTRYCREDENTIAL = 'dockerhub'
        dockerImage = ''
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t ${REGISTRY}:${DOCKER_TAG}"
            }
        }
        stage('Docker Push') {
            steps{
                script {
                    docker.withRegistry( '', REGISTRYCREDENTIAL ) {
                    sh "docker push ${REGISTRY}:${DOCKER_TAG}"
                    }
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
               sh "chmod +x changeTag.sh"
               sh "./changeTag.sh ${DOCKER_TAG}"
               sshagent(['k8s-cicd']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml pods.yml admin@54.252.219.55:/home/admin"
                    script {
                        try {
                            sh "ssh admin@54.252.219.55 kubectl apply -f ."
                        }catch(error){
                            sh "ssh admin@54.252.219.55 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
