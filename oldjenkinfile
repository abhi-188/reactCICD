pipeline {
    environment {
        registry = "habhi/react_devops"
        DOCKER_CREDENTIALS = 'Docker_ID'
        dockerImage = ''
        K8S_CREDENTIALS = 'kubeconfig'
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

        stage('Deploy on k8s'){
            steps{
                withKubeConfig(
                    credentialsId: K8S_CREDENTIALS,
                    caCertificate: '',
                    clusterName: 'minikube',
                    namespace: 'default',
                    serverUrl: ''
                ){
                    sh 'kubectl get pods'
                    //sh 'kubectl delete -f react-deployment.yml'
                    //sh 'kubectl delete -f react-svc.yml'
                    sh 'kubectl apply -f react-deployment.yml'
                    sh 'kubectl apply -f react-svc.yml'
                    sh 'kubectl get svc'
                    sh 'kubectl get deploy'
                    sh 'kubectl get pods'
                }
            }   
        }
    }
}