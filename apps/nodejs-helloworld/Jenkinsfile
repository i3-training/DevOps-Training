pipeline{
  environment {
        TAG = "latest"
        DOCKERHUB_CREDENTIALS=credentials('dockerhub_credentials')
        DOCKER_HUB_REPO = "imransetiadi22/hello-world-nodejs"
  }
  agent any
    stages {
        stage('Build'){
            steps{
                echo 'Building Nodejs..'
                sh 'npm install'
            }
        }
        stage('Build Docker Images') {
            steps {
                echo 'Building..'
                sh 'docker image build -t $DOCKER_HUB_REPO:${TAG} .'
            }
        }
        stage('Push Images to Docker Hub') {
            steps {
                echo 'Pushing image..'
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push $DOCKER_HUB_REPO:${TAG}'
            }
        }
        stage('Deploy to Cluster Kubernetes') {
            steps {
                echo 'Deploying....'
                sh 'sudo kubectl apply -f deployment.yaml'
            }
        }
    }
}

