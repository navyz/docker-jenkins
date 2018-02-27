FROM ubuntu
RUN apt-get update
RUN apt-get install --assume-yes  apt-transport-https ca-certificates curl software-properties-common
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
RUN apt-get update
RUN apt-cache search docker
RUN apt-get install --assume-yes docker-ce
RUN service docker start
RUN ps -ef | grep docker
RUN ps -ef 
RUN ps -ef
RUN ps 
RUN pwd
RUN cd
RUN mkdir .ssh
RUN chmod 0700 .ssh
RUN cd .ssh
RUN apt-get install --assume-yes openssh-server
RUN service ssh restart
RUN apt-get install --assume-yes vim
RUN apt-get install --assume-yes openjdk-8-jdk

# Add jenkins user
RUN adduser  --home /var/lib/jenkins -shell /bin/sh --disabled-password --disabled-login --gecos "" jenkins
RUN echo "PS1='\e[0,31m$\H - \e[32;1m\u@\h: \e[32m\W\e[0m\$ '" >> ~/.bashrc

USER jenkins
WORKDIR /var/lib/jenkins
RUN mkdir /var/lib/jenkins/.ssh
RUN echo "PS1='\e[0,31m$\H - \e[32;1m\u@\h: \e[32m\W\e[0m\$ '" >> ~/.bashrc 
RUN usermod -a -G docker jenkins
RUN usermod -a -G root jenkins
RUN echo 'jenkins ALL=(ALL:ALL) ALL' >> /etc/sudoers

USER root
CMD cp /var/lib/jenkins-share/id* /var/lib/jenkins/.ssh/ && \
	cat /var/lib/jenkins/.ssh/id_rsa.pub >> /var/lib/jenkins/.ssh/authorized_keys && \
	service docker start && service ssh start && \
	chown jenkins:jenkins /var/lib/jenkins/.ssh/* && \
	/bin/bash 

#build: docker build . -t sonlmz/jenkins-worker
#run :  docker run -it --rm  --name jenkins-worker -h jenkins-worker -v /var/run/docker.sock:/var/run/docker.sock --volumes-from jenkins-keys sonlmz/jenkins-worker

