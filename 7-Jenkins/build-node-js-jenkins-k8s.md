## How to CI/CD a Nodejs application using Jenkins as an integration server
The pipeline actually consists of build, test, build a docker image, and push to Dockerhub, Deploy into k8s. Since we did not write any test cases for our app, let us skip it.
Let us first install Jenkins.
Run the Jenkins service file. The first time it asks for an adminpassword which can be found at jenkins/secrets folder.

Then in the next step install suggested plugins.
After installing, the dashboard opens up. Go to Manage Jenkins and select Manage Plugins.

- Install plugins for Nodejs, Docker, Kubernetes, and GitHub.
1. Click button manage jenkins
2. Click manage plugins

- Go to configure system in Manage Jenkins.


