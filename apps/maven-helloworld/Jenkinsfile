pipeline {
    agent any
	
    tools {
        maven 'mvn-3.6.3' 
    }
    environment {
        TAG = "latest"
	DOCKERHUB_CREDENTIALS=credentials('dockerhub_credentials')
        DOCKER_HUB_REPO = "imransetiadi22/hello-world-maven"
    }
    stages {
        stage ('Build Java using Maven') {
            steps {
                sh 'mvn clean package'
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
                sh 'sudo kubectl apply -f service.yaml'
            }
        }
    }
}
