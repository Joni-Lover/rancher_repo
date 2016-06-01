# This Dockerfile is used to build an image containing basic
# configuration to be used as a Jenkins slave build node.

FROM centos:centos7
MAINTAINER ivan_palanevich@gmail.com

ENV container docker
ENV USER jenkins
USER root

RUN yum -y swap -- remove fakesystemd -- install systemd systemd-libs ; \
    yum -y update; yum clean all; \
(cd /lib/systemd/system/sysinit.target.wants/; for i in *; do [ $i == systemd-tmpfiles-setup.service ] || rm -f $i; done); \
rm -f /lib/systemd/system/multi-user.target.wants/*; \
rm -f /etc/systemd/system/*.wants/*; \
rm -f /lib/systemd/system/local-fs.target.wants/*; \
rm -f /lib/systemd/system/sockets.target.wants/*udev*; \
rm -f /lib/systemd/system/sockets.target.wants/*initctl*; \
rm -f /lib/systemd/system/basic.target.wants/*; \
rm -f /lib/systemd/system/anaconda.target.wants/*;

# Install Java 8
RUN yum install -y java-1.8.0-openjdk.x86_64 git subversion gcc gcc-c++ e2fsprogs btrfs-progs iptables xz sudo

# Install Docker from Docker Inc. repositories.
RUN curl -sSL https://get.docker.com/ | sh ; \
    mv /usr/bin/docker /usr/bin/docker.orig ; \
    echo 'DOCKER_HOST=unix:///var/run/docker.sock /usr/bin/docker.orig $@' > /usr/bin/docker ; \
    chmod +x /usr/bin/docker

# Add user & group "$USER" to the image and set its password
RUN groupadd --gid=1003 $USER ; \
    useradd --uid=1003 --gid=1003 -m -g $USER -s /bin/bash $USER ; \
    echo "$USER:$USER" | chpasswd

VOLUME [ "/sys/fs/cgroup" ]
VOLUME /var/lib/docker

ENV SWARM_CLIENT_VERSION 2.0
ADD http://maven.jenkins-ci.org/content/repositories/releases/org/jenkins-ci/plugins/swarm-client/${SWARM_CLIENT_VERSION}/swarm-client-${SWARM_CLIENT_VERSION}-jar-with-dependencies.jar /usr/share/jenkins/swarm-client-${SWARM_CLIENT_VERSION}.jar
RUN chmod 644 /usr/share/jenkins/swarm-client-${SWARM_CLIENT_VERSION}.jar

USER jenkins
WORKDIR /home/jenkins/

ADD ./run.sh /run.sh
ADD ./wrapdocker /usr/local/bin/wrapdocker
RUN chmod +x /usr/local/bin/wrapdocker

# Make sure that the "jenkins" user from evarga's image is part of the "docker"
# group. Needed to access the docker daemon's unix socket.
RUN usermod -a -G docker $USER
RUN echo "%docker ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/docker

ENTRYPOINT ["/run.sh"]
