pipeline {
    environment {
        registry = "habhi/react_devops"
        DOCKER_CREDENTIALS = 'Docker_ID'
        dockerImage = ''
    }
    agent any
    stages {
        stage("Build") {
            steps {
                sh "npm install"
                sh "npm run build"
            }
        } 

        stage('Stop Previous container'){
            steps{
                script{
                    echo '-------------------------------Stoppig Previous container-----------------------'
                    def container_name = 'react_devops'
                    if("${BUILD_NUMBER}" > 0){
                        sh "docker rm --force ${container_name} "
                    }
                    else{
                        echo "--------------------------No running container-----------------------------"
                    }
                }
            }
        } 

        stage('Building our image') {
            steps{
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }

        stage('Deploy our image') {
            steps{
                script {
                    docker.withRegistry( '', DOCKER_CREDENTIALS ) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Cleaning up') {
            steps{
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('Deploy on k8s')
        steps{
            script{
                kubernetesDeploy(configs: "react-deployment.yml","react-svc.yml")
            }
        }
    }
}