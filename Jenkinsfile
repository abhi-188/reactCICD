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
                    namespace: 'default',                        serverUrl: ''
                ){
                    script{
                        def deploymentExists = sh(
                            script:'kubectl get deploy react-devops-deployment -o name',
                            returnStatus: true
                        ) == 0
                        def serviceExists = sh(
                            script:'kubectl get svc react-devops-service -o name',
                            returnStatus: true
                        ) == 0

                        if(deploymentExists){
                            sh 'kubectl delete -f react-deployment.yml'
                        }
                        if(serviceExists){
                            sh 'kubectl delete -f react-svc.yml'
                        }
                        
                        sh 'kubectl apply -f react-deployment.yml'
                        sh 'kubectl apply -f react-svc.yml'

                        sh 'sleep 10'

                        sh 'kubectl get svc'
                        sh 'kubectl get deploy'
                        sh 'kubectl get pods'
                    }
                } 
            }  
        }
    }
}