FROM alpine:3.3

ENV PACKAGES "git openssh-client gnupg bash"

RUN apk add --update $PACKAGES && rm -rf /var/cache/apk/*

RUN mkdir -p /root/.ssh

RUN git config --global user.email "git-ssh@governmentpaas.docker" && \
    git config --global user.name "Docker container git-ssh"

# Create share folder which contains all the ssh keys required for ssh between containers
RUN mkdir /var/lib/jenkins-dockshare

# Add jenkins user

RUN adduser  -h /var/lib/jenkins -s /bin/sh -D jenkins
RUN chown jenkins:jenkins /var/lib/jenkins-share

USER jenkins
WORKDIR /var/lib/jenkins

# Gen key and add to share directory
RUN ssh-keygen -b 2048 -t rsa -f /var/lib/jenkins/.ssh/id_rsa -q -N ""
RUN cp /var/lib/jenkins/.ssh/id* /var/lib/jenkins-share/
RUN echo "PS1='\e[0,31m$\H - \e[32;1m\u@\h: \e[32m\W\e[0m\$ '" >> ~/.bashrc 

CMD /bin/bash

# BULD
#docker build . -t sonlmz/jenkins-keys

# RUN
#docker run -it --name jenkins-keys -h jenkins-keys -v /var/lib/jenkins-share -e sonlmz/jenkins-keys


