pipeline {
    environment {
        registry = "habhi/react_devops"
        registryCredential = 'Docker_ID'
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
                    docker.withRegistry( '', registryCredential ) {
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
    }
}