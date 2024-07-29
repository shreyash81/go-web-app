pipeline{
    agent{
        docker{
            image 'golang:1.21'
            args '-v /var/run/docker.sock:/var/run/docker.sock --user root'
        }
    }
  stages{
    stage('checkout'){
        steps{
            echo 'checking out the code...'
            checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: 'https://github.com/shreyash81/go-web-app.git']]])
        }

    }

    stage('Install Docker'){
        steps{
            echo 'Installing Docker...'
            sh '''
             curl -fsSL https://get.docker.com -o get-docker.sh
                    sh get-docker.sh
            '''
        }

    }
    stage('Install Dependencies'){
        steps{
            echo 'Installing the Depemdencies...'
            sh 'go mod download'
        }
    }
    stage('Build the Project'){
        steps{
            echo 'Building the Project or Artifact'
            sh 'go build -o main'
        }
    }
    stage('Build and Push the Docker Image'){
        environment{
            REGISTRY_CREDENTIALS = credentials('docker-cred')
            DOCKER_IMAGE = "chauhanshreyash18/ultimate-golang-cicd:${BUILD_NUMBER}"
            DOCKERFILE_LOCATION = "./DOCKERFILE"

        }
        steps{
            echo 'building the docker image and then pushing to docker registry'
            script{
                sh "docker build -t ${DOCKER_IMAGE} -f ${DOCKERFILE_LOCATION} ."
                sh "docker tag ${DOCKER_IMAGE} index.docker.io/${DOCKER_IMAGE}"

                withCredentials([usernamePassword(credentialsID: 'docker-cred' , usernameVariable: 'DOCKER_USERNAME' , passwordVariable: 'DOCKER_PASSWORD')]){
                    sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin"
                    sh "docker push index.docker.io/${DOCKER_IMAGE}"
                }

            }
        }
    }
    stage('Update the Deployment'){
        environment{
            GIT_REPO_NAME = "go-web-app"
            GIT_USER_NAME = "shreyash81"
        }
        steps{
            echo " Updating the manifest..."
            withCredentials([string(credentialsID: 'github' , variable: "GITHUB_TOKEN")])
              sh '''
                    git config user.email "chauhanshreyash357@gmail.com"
                    git config user.name "Shreyash singh"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" helm/values.yaml
                    git add  helm/values.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''

        }
    }
}
post {
    success{
        echo 'pipeline Succed'
    }
    faliure{
        echo 'pipeline failed'
}
    
}



}
