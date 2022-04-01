pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = "mach512/testflask"
        CONTAINER_NAME = "flask-container"
        STUB_VALUE = "200"
    }
    stages {
        stage('Stubs-Replacement'){
            steps {
                // 'STUB_VALUE' Environment Variable declared in Jenkins Configuration 
                echo "STUB_VALUE = ${STUB_VALUE}"
                sh "sed -i 's/<STUB_VALUE>/$STUB_VALUE/g' config.py"
                sh 'cat config.py'
            }
        }
        stage('Build') {
            steps {
                //  Pushing Image to Repository
                withCredentials([usernamePassword( credentialsId: 'docker-hub-credentials', usernameVariable: 'mach512', passwordVariable: 'dbZZ@2005')]) {
                    def registry_url = "registry.hub.docker.com/"
                    bat "docker login -u $USER -p $PASSWORD ${registry_url}"
                    docker.withRegistry("http://${registry_url}", "docker-hub-credentials") {
                    //  Building new image
                        bat 'docker image build -t $DOCKER_HUB_REPO:latest .'
                        bat 'docker image tag $DOCKER_HUB_REPO:latest $DOCKER_HUB_REPO:$BUILD_NUMBER'
                    // Push your image now
                        bat "docker push mach512/testflask:$BUILD_NUMBER"
                        bat "docker push mach512/testflask:latest"
                        echo "Image built and pushed to repository"
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                script{
                    //sh 'BUILD_NUMBER = ${BUILD_NUMBER}'
                    if (BUILD_NUMBER == "1") {
                        sh 'docker run --name $CONTAINER_NAME -d -p 5000:5000 $DOCKER_HUB_REPO'
                    }
                    else {
                        sh 'docker stop $CONTAINER_NAME'
                        sh 'docker rm $CONTAINER_NAME'
                        sh 'docker run --name $CONTAINER_NAME -d -p 5000:5000 $DOCKER_HUB_REPO'
                    }
                    //sh 'echo "Latest image/code deployed"'
                }
            }
        }
    }
}
