FROM alpine:3.3

ENV PACKAGES "openssh-client gnupg bash"

RUN apk add --update $PACKAGES && rm -rf /var/cache/apk/*

RUN mkdir -p /root/.ssh

# Add jenkins user
RUN adduser  -h /var/lib/jenkins -s /bin/sh -D jenkins


USER jenkins
WORKDIR /var/lib/jenkins
RUN mkdir /var/lib/jenkins/.ssh

#copy keys
RUN ls -la /var/lib/jenkins/
RUN echo "PS1='\e[0,31m$\H - \e[32;1m\u@\h: \e[32m\W\e[0m\$ '" >> ~/.bashrc 

CMD cp /var/lib/jenkins-share/id* /var/lib/jenkins/.ssh/ && \
	cat /var/lib/jenkins/.ssh/id_rsa.pub >> /var/lib/jenkins/.ssh/authorized_keys && \
	/bin/bash


# BULD
#docker build . -t sonlmz/jenkins-data

# RUN
#docker run -it --name jenkins-data -h jenkins-data -v /var/lib/jenkins --volumes-from jenkins-keys -e ROLE=slave sonlmz/jenkins-data

