# docker-jenkins

# SpringBoot + Gradle + Docker + Jenkins
## Preface
- This tutorial guide you to use gradle to build a SpringBoot application to be a docker images. Jenkins is configured to automate the task
- This setup is for developer, all the script to build the application, docker code, ... is all available here. 
- The docker images is setup in the way what easy for you to tweet, not optimize on the sizing. That's your part.... And as I said before, all the code is available for you.
- 
## Pre-require
To run this you need the following software install
- docker
- java8
- and a console to run your commands
All the other software required will be installed inside the docker.

## Master steps
In general, below are the steps to setup and run this demo
1. Test to build the project by your own.
2. Build the docker images to automate your tasks
3. docker run ...

## Test to build the project
1. Pull the source code

``` shell
git pull https://github.com/navyz/docker-jenkins.git
```

3. Build

``` shell
cd docker-jenkins/myWeb
./gradlew clean build docker
```

The docker images should be build for you. Run the command 
``` shell
docker images
```
The new docker images "sonlmz/myweb" should be listed in there

4. Run the application
``` shell
docker run -it --rm -p 8080:8080 sonlmz/myweb
```
Open your browser and access to the address http://localhost:8080, the "Welcome to springboot + gradle + docker + jenkins" should be there!

5. Clean up
Now, we are all know that we can use the gradle to build the docker for our web application. Now we delete it, and let the jenkins will do that for you.
It's a long story, but we just do step by step until we can see Jenkins do all the stuff for us, automatically.

## Build the docker images

- Technically, we can build everything in one docker images. But, for the sake of beauty, we need 4 docker images to build one final docker image (which is myweb). That's sound ridiculous, but please trust me, it's beauty.
1. jenkins 
2. jenkins-worker
3. jenkins-keys
4. jenkins-data

- jenkins: The first docker images seem obvious, we want jenkins to build the application for us, then we need him. 
- jenkins-worker: when jenkins need to build an application, jenkins don't build by itself but  it will ask another guys to do for him. And here we come jenkins-work. We only have one jenkins instance running but we can have many jenkins-work created. Let say, we have 100 applications need to be built (by jenkins), some using maven 3, other uing maven 4, some use nodejs4, and another use nodejs5. We will setup multiple worker, and each will have the corresponding software:version needed.
- jenkins-keys: since we have multiple docker containers running, they need to communicate with each other, this instance will take care of create the ssh keys and share for all of them.
- jenkins-data: hold all the jenkins configuration (persistency). If we want to restart jenkins, or update software, ... all the configuration we created is still there for you.

May be you don't understand much about the above explanations, just keep going on. We will know sooner or later.

### jenkins images
The jenkins image is availabe at sonlmz/jenkins. You can just pull and run anytime you want
``` shell
docker run -it -p 8080 sonlmz:jenkins
```
The jenkins will then can be accessed via http://localhost:8080/
But don't do it for now. Because we don't want just a jenkins running alone, we want a set up an eco-system which 4 docker containers as mentioned above. 
> If you accidentally run it, you can just simple remove it for now

If you don't want to use the pre-built docker images, you can just build your self. Back to the folder we just download from git in the previous step (git pull https://github.com/navyz/docker-jenkins.git)

``` shell
cd docker-jenkins/jenkins
docker build . sonlmz/jenkins
```
 And then the sonlmz/jenkins image is there in your local docker store. The different is that, it will pull the latest dependency image and software. Even those latest is good, I can' guarantee that you can run it smoothly because there is may be there is a break-changes somewhere here and there. Before you are familiar with all of this, I still recommend you to just use the built-image, except that you have some other good reason.
 ### jenkins-worker

... being updating 
