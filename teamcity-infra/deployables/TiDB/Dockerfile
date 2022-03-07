FROM registry.redhat.io/ubi8

#Copy MariaDB Repo
COPY ./mariadb.repo /etc/yum.repos.d/

# Install the required software
RUN yum update -y && yum install nmap tar curl -y

#Copy tar file from local
COPY oc.tar /tmp/

#Get oc file
RUN curl -o /tmp/oc.tar https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/oc/latest/linux/oc.tar.gz; exit 0

#Untar oc file
RUN tar -xzvf /tmp/oc.tar -C /tmp/ && mv /tmp/oc /usr/local/bin
