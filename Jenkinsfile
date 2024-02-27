pipeline {
     agent any

    environment {
        DOCKER_CREDENTIALS = credentials('Docker_ID')
        DOCKER_IMAGE_NAME = 'habhi/react_devops'
        DOCKER_IMAGE_TAG = 'latest'
    }

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

        stage('Remove old Image') {
            steps {
                script { 
                    def imageName = "habhi/react_devops"
                    env.imageName = "${imageName}"
                    echo "${env.imageName}"
                    def oldImageID = sh( 
                                            script: 'docker images -qf reference=\${imageName}:latest',
                                            returnStdout: true
                                        )

                    echo "Image Name: " + "${imageName}"
                    echo "Old Image: ${oldImageID}"

                    if ( "${oldImageID}" != '' ) {
                        echo "Deleting image id: ${oldImageID}..."
                        sh "docker rmi -f ${oldImageID}"
                    }
                    else {
                        echo "No image to delete..."
                        } 
                    }  
                }
            }


        stage('Creating Image of build') {
            steps{
                script {
                    echo '---------------------------Building Image----------------------------------'
                    sh 'docker build -t habhi/react_devops .'
                    //build will serach for dockerfile in repository
                    echo '---------------------------Image Successfully Build---------------------------------'
                }
            }
        }

        

        stage('Pushing image to DockerHub') {
            steps{
                script{
                    echo '-----------------------------Pushing Image----------------------------------------'
                    /* groovylint-disable-next-line NestedBlockDepth */
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        //docker.image(DOCKER_IMAGE_NAME).push(DOCKER_IMAGE_TAG)
                        sh 'docker push habhi/react_devops'
                        echo '-------------------------Image Successfully pushed--------------------------------'
                    }
                } 
            }
        }

        

        stage('Production(Running Container)'){
            steps{
                script{
                    echo '-----------------------------Running Container-------------------------------------'
                    sh 'docker pull habhi/react_devops:latest'
                    sh 'docker run --name react_devops -d -p 80:80 habhi/react_devops:latest' 
                }   
            }
        }
        /*
        stage('K8s Deployment'){
            steps{
                // script{
                //     // sh '/usr/local/bin/kubectl apply -f ~/react-svc.yml'
                //     sh '/usr/local/bin/kubectl get nodes'

                // }
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'K8S', namespace: '', serverUrl: 'https://38DB0FEA005008A7C1A1D14B3555B550.gr7.us-east-1.eks.amazonaws.com') {
                    // sh 'curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.16.8/2020-04-16/bin/linux/amd64/kubectl'
                    // sh 'chmod u+x ./kubectl'
                    //sh '/usr/local/bin/kubectl get nodes'
                    sh 'kubectl get nodes'
                    // sh 'kubectl apply -f react-svc.yml'
                }
            }
        }
        */
            
    }
}