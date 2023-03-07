# Running Pipeline with Freestyle Job

A Freestyle pipeline in Jenkins is a type of Jenkins project that provides a configurable interface for building software projects. It allows developers to configure various build steps and settings, such as source code management, build triggers, build environment variables, build scripts, and post-build actions. The Freestyle project provides a wide range of flexibility and can be used for simple and complex projects alike. The pipeline view can be used to visualize the entire build process.

## 1. Create Freestyle Project

Create freestyle pipeline click on New Item, Give your pipeline a name and then select Freestyle Project as shown by picture below.
<img width="271" alt="1" src="https://user-images.githubusercontent.com/22531977/223299957-04a8f774-3bff-4d90-959d-ee8326337cc0.png">
<img width="1280" alt="2" src="https://user-images.githubusercontent.com/22531977/223299980-8a421d67-ab66-4f77-8f8c-3a9fa464d6ab.png">

## 2. Configure Build

Couple of value we need to add for this pipeline as listed below,

1. Description field so we can easily recognize the propose of this pipeline.

    

2. Source Code Management field, here we need to add our source code hosted on git, if your source code in a private repository, you need to give credential so jenkins can pull the repository, on Branches to build select your desire branch.
<img width="1280" alt="3" src="https://user-images.githubusercontent.com/22531977/223300168-7b3a7c8f-e733-4932-9a88-8352d4962f6d.png">

    

3. Build Steps is where we define the way our application si build. in this example the pipeline will be separate to 3 step. First step is to build the application and package it as container image.
<img width="1280" alt="4" src="https://user-images.githubusercontent.com/22531977/223300272-7f865071-7b19-4357-8055-8659d5f8f891.png">


```bash
echo "BUILD CONTAINER IMAGE" 
docker build --pull --rm -f "app/go-sample/Dockerfile" -t go-app:sample1 "app/go-sample"
```

Second step is to run the image on the vm itself and expose the application to outside network.

```bash
echo "RUN IMAGE AS CONTAAINER"
docker run --name gorest --rm -d -p 80:80/tcp -p 9090:9090/tcp go-app:sample1
```

Third step is using `curl` to access the application and see if the application is running correctly.

```bash
echo "ACCESS TO CHECK APP"
curl 127.0.0.1:9090/articles
curl 127.0.0.1:9090/article/1
```

Apply and save your project configuration.

## 3. Run Your Pipeline
<img width="1280" alt="5" src="https://user-images.githubusercontent.com/22531977/223300337-fc78c606-7238-4056-99ad-ed2378e988d2.png">
<img width="1280" alt="6" src="https://user-images.githubusercontent.com/22531977/223300345-eeeaa1db-a39a-4dd0-b8f3-07c1584006c6.png">
<img width="1280" alt="image" src="https://user-images.githubusercontent.com/22531977/223300991-731b84e6-b2ac-40e7-921c-a88af96f9191.png">



After you save the project configuration you will be redirected to project overview, Select Build Now to run the project, when project is running you can inspect build process and find details regarding build itself, from here you can troubleshot if theres issue during build, image below show of how build process running and its logs.

