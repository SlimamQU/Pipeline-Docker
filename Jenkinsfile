def CONTAINER_NAME="jenkins-pipeline"
def CONTAINER_TAG="latest"
def DOCKER_HUB_USER="anujsharma1990"
def HTTP_PORT="8090"

pipeline {
    agent any
    stages{
        stage('Initialize'){
            steps{
                script{
                    def dockerHome = tool 'myDocker'
                    def mavenHome  = tool 'myMaven'
                    env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
                }
            }
        }

        stage('Checkout') {
            steps{
                checkout scm
            }
        }

        stage('Build'){
            steps{
                sh "mvn clean install" 
            }
        }

        stage('Sonar'){
            steps{
                script{
                    try {
                        sh "mvn sonar:sonar"
                    } catch(error){
                        echo "The sonar server could not be reached ${error}"
                    }
                }
            }
        }

        stage("Image Prune"){
            steps{
                imagePrune(CONTAINER_NAME)
            }
        }

        stage('Image Build'){
            steps{
                imageBuild(CONTAINER_NAME, CONTAINER_TAG)
            }
        }

        stage('Push to Docker Registry'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
                }
            } 
        }

        stage('Run App'){
            steps{
                runApp(CONTAINER_NAME, CONTAINER_TAG, DOCKER_HUB_USER, HTTP_PORT)
            }
        }
    }
}

def imagePrune(containerName){
    try {
        sh "sudo docker image prune -f"
        sh "sudo docker stop $containerName"
    } catch(error){}
}

def imageBuild(containerName, tag){
    sh "sudo docker build -t $containerName:$tag  -t $containerName --pull --no-cache ."
    echo "Image build complete"
}

def pushToImage(containerName, tag, dockerUser, dockerPassword){
    sh "sudo docker login -u $dockerUser -p $dockerPassword"
    sh "sudo docker tag $containerName:$tag $dockerUser/$containerName:$tag"
    sh "sudo docker push $dockerUser/$containerName:$tag"
    echo "Image push complete"
}

def runApp(containerName, tag, dockerHubUser, httpPort){
    sh "sudo docker pull $dockerHubUser/$containerName"
    sh "sudo docker run -d --rm -p $httpPort:$httpPort --name $containerName $dockerHubUser/$containerName:$tag"
    echo "Application started on port: ${httpPort} (http)"
}
