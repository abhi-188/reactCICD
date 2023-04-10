pipeline {
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
                    docker.withRegistry('', '83626bcd-d2fa-4b05-83e0-1a4436f8804d') {
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
        // stage("Deploy") {
        //     steps {
        //         sh "rm -rf /var/www/react_app"
        //         sh "cp -r ${WORKSPACE}/build/ /var/www/react_app/"
        //     }
        // }
            
    }
}