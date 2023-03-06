# How to CI/CD a Nodejs application using Jenkins as an integration server
The pipeline actually consists of build, test, build a docker image, and push to Dockerhub, Deploy into k8s. Since we did not write any test cases for our app, let us skip it.
Let us first install Jenkins.
Run the Jenkins service file. The first time it asks for an adminpassword which can be found at jenkins/secrets folder.

Then in the next step install suggested plugins.
After installing, the dashboard opens up. Go to Manage Jenkins and select Manage Plugins.

## - Install plugins for Nodejs, Docker.
1. Click button manage jenkins
2. Click manage plugins
<img width="1280" alt="1" src="https://user-images.githubusercontent.com/22531977/223068275-5b28f2a2-cd5f-44cd-a6f4-da62c1e1b6d5.png">
and then search on menu available plugins "docker" click install without restart 
<img width="1280" alt="2" src="https://user-images.githubusercontent.com/22531977/223071224-12c7a5b0-28f2-4848-af41-52c843c671dc.png">
next search on menu available plugins "nodejs" click install without restart 
<img width="1280" alt="3" src="https://user-images.githubusercontent.com/22531977/223071613-bfcb8612-ffc1-4796-92d7-9e6d50d3fae9.png">
3. Go to dashboard > manage jenkins > global tool configuration, to setting docker installation & nodejs installation
<img width="1280" alt="global" src="https://user-images.githubusercontent.com/22531977/223088074-39f61654-c3dc-4487-bb05-5dd31b289309.png">
<img width="1280" alt="docker" src="https://user-images.githubusercontent.com/22531977/223088143-30ee2148-985b-4afc-920c-5857c07a62f8.png">
<img width="1280" alt="nodejs" src="https://user-images.githubusercontent.com/22531977/223088151-0fe34615-7c61-464a-801b-7cfa17dea685.png">
4. Go to Dashboard > manage jenkins > manage credentials to setup username and password github (repo source code) and dockerhub credentials
<img width="1280" alt="4" src="https://user-images.githubusercontent.com/22531977/223073585-52206202-85e8-4a0f-86e1-29e72a4b28ed.png">
Click add credentials
<img width="1280" alt="5" src="https://user-images.githubusercontent.com/22531977/223077589-c6a93eac-fdd0-4631-b560-93762052e89c.png">
<img width="1280" alt="5 5" src="https://user-images.githubusercontent.com/22531977/223077297-12b474a1-504a-4a0b-a7de-3a2b618fd24b.png">
fill column with username and password github for repository source code
<img width="1280" alt="6" src="https://user-images.githubusercontent.com/22531977/223077833-28fe77c2-14d1-4590-94de-3a9870b2879f.png">
fill column with username and password dockerhub for repository docker images
<img width="1280" alt="7" src="https://user-images.githubusercontent.com/22531977/223077968-e6f97d28-893e-4256-8ab2-548b72bd20dd.png">

## - Now let's create a pipeline.
```sh
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
```
Let’s breakdown the Jenkinsfile
Firstly, We are setting the environment variable to be used in the file.
The registry is the docker hub registry where we want to push the image and the image name.
registry credentials are of the docker hub. The docker hub credentials id which we have saved in Jenkins a while ago.
Then we have specified the stages of our pipeline and what each stage has to do in the pipeline.
Push this file into your Github-repository.
Now lets create a pipeline project in Jenkins.

Back to dashboard Jenkins > Click on new Item, Put any name and choose pipeline and save it.
<img width="1280" alt="8" src="https://user-images.githubusercontent.com/22531977/223080062-7d49a331-3cac-4556-9819-776b038e28b4.png">
<img width="1280" alt="9" src="https://user-images.githubusercontent.com/22531977/223080492-fc81a847-ec4e-4546-b1a4-e1dc71a734a6.png">
after create new item, go to menu pipeline, setting pipeline script with scm, repository url, credentials github, branch to build code and script path is Jenkinsfile, next click save
<img width="1280" alt="11" src="https://user-images.githubusercontent.com/22531977/223083808-7c2ae8ed-8256-4cde-a6a1-eb5526f80524.png">
Next, click build now to running pipeline 
<img width="1280" alt="12" src="https://user-images.githubusercontent.com/22531977/223086439-385261a6-7a3b-4800-8873-f90af77613a4.png">







