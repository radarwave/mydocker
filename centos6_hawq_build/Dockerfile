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
 openssh-clients openssh-server bison bison-devel libffi-devel && \
 yum clean all

RUN cd /tmp; curl https://cmake.org/files/v3.1/cmake-3.1.0-Linux-x86_64.tar.gz -o cmake-3.1.0-Linux-x86_64.tar.gz && \
  tar xzvf cmake-3.1.0-Linux-x86_64.tar.gz && cd /tmp/cmake-3.1.0-Linux-x86_64 && sudo cp -rf * /usr/

# Install apache maven
RUN curl -L "http://repos.fedorapeople.org/repos/dchen/apache-maven/epel-apache-maven.repo" -o /etc/yum.repos.d/epel-apache-maven.repo && \
 yum install -y apache-maven && \
 yum clean all

# Install python modules
RUN pip --retries=50 --timeout=300 install pg8000 simplejson unittest2 pycrypto pyyaml lockfile paramiko psi && \
  pip --retries=50 --timeout=300 install http://darcs.idyll.org/~t/projects/figleaf-0.6.1.tar.gz && \
  pip --retries=50 --timeout=300 install http://sourceforge.net/projects/pychecker/files/pychecker/0.8.19/pychecker-0.8.19.tar.gz/download

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

RUN echo $'* soft nofile 2900000\n\
* hard nofile 2900000\n\
* soft nproc 131072\n\
* hard nproc 131072\n' \
 >> /etc/security/limits.conf

RUN echo $'kernel.shmmax = 1000000000\n\
kernel.shmmni = 4096\n\
kernel.shmall = 4000000000\n\
kernel.sem = 250 512000 100 2048\n\
kernel.sysrq = 1\n\
kernel.core_uses_pid = 1\n\
kernel.msgmnb = 65536\n\
kernel.msgmax = 65536\n\
kernel.msgmni = 2048\n\
net.ipv4.tcp_syncookies = 0\n\
net.ipv4.ip_forward = 0\n\
net.ipv4.conf.default.accept_source_route = 0\n\
net.ipv4.tcp_tw_recycle = 1\n\
net.ipv4.tcp_max_syn_backlog = 200000\n\
net.ipv4.conf.all.arp_filter = 1\n\
net.ipv4.ip_local_port_range = 1281 65535\n\
net.core.netdev_max_backlog = 200000\n\
vm.overcommit_memory = 2\n\
fs.nr_open = 3000000\n\
kernel.threads-max = 798720\n\
kernel.pid_max = 798720\n\
net.core.rmem_max=2097152\n\
net.core.wmem_max=2097152\n'\
 > /etc/sysctl.conf

RUN sysctl -p

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
