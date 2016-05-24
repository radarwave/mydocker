FROM centos:6

MAINTAINER Radar Lei <rlei@pivotal.io>

# install document with yum
RUN sed -i 's/tsflags=nodocs//g' /etc/yum.conf

# install all software we need
RUN yum install -y epel-release && \
 yum makecache && \
 yum install -y man passwd sudo tar which git mlocate links make bzip2 \
 autoconf automake libtool m4 gcc gcc-c++ gdb flex cmake gperf indent \
 libuuid-devel krb5-devel libgsasl-devel expat-devel libxml2-devel \
 perl-ExtUtils-Embed pam-devel python-devel snappy-devel \
 libyaml-devel libevent-devel bzip2-devel openssl-devel \
 openldap-devel readline-devel net-snmp-devel apr-devel \
 libesmtp-devel xerces-c-devel python-pip json-c-devel \
 apache-ivy java-1.7.0-openjdk-devel R-core-devel \
 openssh-clients openssh-server bison bison-devel && \
 yum clean all

RUN cd /tmp; curl https://cmake.org/files/v3.1/cmake-3.1.0-Linux-x86_64.tar.gz -o cmake-3.1.0-Linux-x86_64.tar.gz && \
  tar xzvf cmake-3.1.0-Linux-x86_64.tar.gz && cd /tmp/cmake-3.1.0-Linux-x86_64 && sudo cp -rf * /usr/

# Install apache maven
RUN curl -L "http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo" -o /etc/yum.repos.d/epel-apache-maven.repo && \
 yum install -y apache-maven && \
 yum clean all

# Upgrade GCC
RUN cd /etc/yum.repos.d && curl http://people.centos.org/tru/devtools-2/devtools-2.repo -o devtools-2.repo
RUN yum install -y devtoolset-2-gcc devtoolset-2-binutils devtoolset-2-gcc-c++ || pwd
RUN scl enable devtoolset-2 bash && gcc --version
RUN echo "source /opt/rh/devtoolset-2/enable" >> /root/.bashrc

# setup ssh server and keys for root
RUN ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa && \
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
 chmod 0600 ~/.ssh/authorized_keys

# setup JAVA_HOME for all users
RUN echo "#!/bin/sh" > /etc/profile.d/java.sh && \
 echo "export JAVA_HOME=/etc/alternatives/java_sdk" >> /etc/profile.d/java.sh && \
 chmod a+x /etc/profile.d/java.sh

# Remove openjdk client folder
RUN sudo rm -rf /etc/alternatives/java_sdk/jre/lib/amd64/client

# create user gpadmin since HAWQ cannot run under root
RUN groupadd -g 1000 gpadmin && \
 useradd -u 1000 -g 1000 gpadmin && \
 echo "gpadmin  ALL=(ALL)       NOPASSWD: ALL" > /etc/sudoers.d/gpadmin

# sudo should not require tty
RUN sed -i -e 's|Defaults    requiretty|#Defaults    requiretty|' /etc/sudoers

RUN echo "#!/bin/bash" > /etc/profile.d/user.sh && \
 echo "export USER=\`whoami\`" >> /etc/profile.d/user.sh && \
 chmod a+x /etc/profile.d/user.sh

ENV BASEDIR /data
RUN mkdir -p /data && chmod 777 /data

USER gpadmin

RUN echo "source /opt/rh/devtoolset-2/enable" >> ~/.bashrc
# setup ssh client keys for gpadmin
RUN ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa && \
 cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys && \
 chmod 0600 ~/.ssh/authorized_keys

USER root

WORKDIR /data
