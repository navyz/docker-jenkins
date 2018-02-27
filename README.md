# SpringBoot + Gradle + Docker + Jenkins
## Preface
- This tutorial guide you to use gradle to build a SpringBoot application to be a docker images. Jenkins is configured to automate the task.
- The setup is based on appcontainers/jenkins articles. It is a great article/idea/example. However, the author has stopped updating the images a couple of years ago. Some software's versions are getting old. One more thing, I can not find the source code for building the docker images. Eventhose the Dockerfile are there, the config.sh script are not available.
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
docker run -it --rm -p 8081:8080 sonlmz/myweb
```
Open your browser and access to the address http://localhost:8081, the "Welcome to springboot + gradle + docker + jenkins" should be there!

5. Clean up
Now, we are all know that we can use the gradle to build the docker for our web application. Now we delete it, and let the jenkins will do that for you.
It's a long story, but we just do step by step until we can see Jenkins do all the stuff for us, automatically.

## Build the docker images

- Technically, we can build everything in one docker images. But, for the sake of beauty, we need 4 docker images to build one final docker image (which is myweb). That's sound ridiculous, but please trust me, it's beauty.
1. jenkins 
2. jenkins-worker
3. jenkins-keys
4. jenkins-data

> The docker images are pre-built for you. If you don't want to re-built, then just skip this section and go to the next one "Run the application". Or if you want to understand how it works under the hook, then you can read to understand more.

### Jenkins images
The first docker images seem obvious, we want jenkins to build the application for us, then we need him. 
The jenkins image is availabe at sonlmz/jenkins. 

Pre-built docker images
``` shell
docker pull sonlmz/jenkins
```

Docker file (source) - **only if you want to rebuild the image **
If you don't want to use the pre-built docker images, you can just build your self. Back to the folder we just download from git in the previous step (git pull https://github.com/navyz/docker-jenkins.git)

``` shell
git pull https://github.com/navyz/docker-jenkins.git (if you have not pull)
cd docker-jenkins/jenkins
docker build . sonlmz/jenkins
```
 And then the sonlmz/jenkins image is there in your local docker store. The different is that, it will pull the latest dependency image and software. Even those latest is good, I can' guarantee that you can run it smoothly because there is may be there is a break-changes somewhere here and there. Before you are familiar with all of this, I still recommend you to just use the built-image, except that you have some other good reason.

### jenkins-worker
When jenkins need to build an application, jenkins don't build by itself but  it will ask another guys to do for him. And here we come jenkins-work. We only have one jenkins instance running but we can have many jenkins-work created. Let say, we have 100 applications need to be built (by jenkins), some using maven 3, other using maven 4, some use nodejs4, and another use nodejs5. We will setup multiple worker, and each will have the corresponding software:version needed.

Pre-built docker images
``` shell
docker pull sonlmz/jenkins-worker
```
Docker file (source) - **only if you want to rebuild the image **
``` shell
git pull https://github.com/navyz/docker-jenkins.git (if you have not pull)
cd docker-jenkins/jenkins-worker
docker build . sonlmz/jenkins-worker
```

### jenkins-keys
Since we have multiple docker containers running, they need to communicate with each other, this instance will take care of create the ssh keys and share for all of them.


Pre-built docker images
``` shell
docker pull sonlmz/jenkins-keys
```
Docker file (source) - **only if you want to rebuild the image **
``` shell
git pull https://github.com/navyz/docker-jenkins.git (if you have not pull)
cd docker-jenkins/jenkins-keys
docker build . sonlmz/jenkins-keys
```

### jenkins-data
Hold all the jenkins configuration (persistency). If we want to restart jenkins, or update software, ... all the configuration we created is still there for you.


Pre-built docker images
``` shell
docker pull sonlmz/jenkins-data
```
Docker file (source)
``` shell
git pull https://github.com/navyz/docker-jenkins.git (if you have not pull)
cd docker-jenkins/jenkins-data
docker build . sonlmz/jenkins-data
```
Open your browser and access to the address http://localhost:8081, the "Welcome to springboot + gradle + docker + jenkins" should be there!


## Run application
### Run containers

``` shell
docker run -it --name jenkins-keys -h jenkins-keys -v /var/lib/jenkins-share -e sonlmz/jenkins-keys

docker run -it --name jenkins-data -h jenkins-data -v /var/lib/jenkins --volumes-from jenkins-keys -e ROLE=slave sonlmz/jenkins-data

docker run -it --rm  --name jenkins-worker -h jenkins-worker -v /var/run/docker.sock:/var/run/docker.sock --volumes-from jenkins-keys sonlmz/jenkins-worker

docker run -it --rm --name jenkins -h jenkins -p 8080:8080 --volumes-from jenkins-keys --volumes-from jenkins-data -e ROLE=slave -e TERMTAG=JENKINS-DATA --link jenkins-worker:jenkins-worker sonlmz/jenkins
```
It takes sometime to start the jenkins containers (30 seconds on my PC). When it's ready, try to access to http://localhost:8080, the web site should be ready and asking for the admin initial password

Open a new console and run the below command
``` shell
docker exec jenkins cat /var/lib/jenkins/secrets/initialAdminPassword
```
Copy the password and paste to your jenkins site. Click to "install recommended plugins". It may takes a couple of minutes for all the plugins to be install. 

### Create a docker worker node
After all the plugins are installed, go back to jenkins home page by clicking Jenkins logo on the top-left -> click Manage Jenkins -> Manage Nodes. On the left menu, click on "New Node" and give the below information
- Node name: worker
> or any name you name it
- Permanent Agent: ticked
- Click OK
On the new form, provide the following information
- Name: worker
- Description: the agent to execute all build command
- # of executes: 3
- Remote directory: /var/lib/jenkins
> this is the jenkins home directory
- Labels: jenkins-worker
> We will refer to this label later when create a build job
- Usage: Only build jobs with label expressions matching this node
- Launch method: Launch slave agents via SSH
- Host: jenkins-worker
> This is the hostname of our docker container. Please type carefully to make sure it match with our container's hostname.
- Credentials: Add a new one with below information
	- Kind: SSH Username with private key
	- Username: jenkins
	- Privatekey: From the Jenkins Master ~/.ssh
	> jenkins-keys have generate the key for us and share to all the docker containers. We don't need to bother much about this key.
	- Then click Add
- Back to the new Node page, select Jenkins from "Credentials" dropbox
- Host Key Verification Strategy: Non verifying Verification Strategy
> This is important, please remember to select, or else jenkins can't ssh to the docker container
Then click Save

Now come back to the "Manage Node" page, click on the new created node, and click on "Launch Agent". Wait until the log show that "Agent successfully connected and online"
Click back to the home page.

### Create a job to build docker image from java application
From jenkins home page, click on New Item. On the new item page, provide the following information
- Project name: myWeb
- Restricted where this project can be run: jenkins-worker
- Source Code Management: Git
- Repositories: https://github.com/navyz/myWeb
> Leave the Credentials blank here because we don't need an account to download the Git repo
- On the build section, click Add build step -> Execute shell
- Type the following to the Command textbox
``` shell
./gradlew clean build docker
```
Click Save
### Trigger the job to build
Back to the home page, myWeb project, and then click on "Build Now" link on the left menu. Wait for a couple seconds then you can see the agent jenkins-docker will be active and build.
Open the console output to view the log result. 
If everything goes well, we will see the message "Finished SCUSSESS"
Back to our console, type the following command:
```
docker images
```
The sonlmz/myweb should be listed in there, among our other jenkins containers.
### Run the just built docker image

```
docker run -it --rm -p 8081:8080 sonlmz/myweb
```

Open your browser and access to the address http://localhost:8081, the "Welcome to springboot + gradle + docker + jenkins" should be here!


