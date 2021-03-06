######################################
# ---   Java-Stand By Astemir97  --- #
######################################
#
FROM centos:centos8
#
MAINTAINER Devops-eng.
#
#
# --------  Label  ---------
#
LABEL Stand="Dev" \
      Name="Mortgage/CentOS-8" \
      Version="1.0"
#
#
# ---------  Expose  ---------
#
EXPOSE 22 80
#
#
# ---------  Update CentOS  ---------
#
RUN yum update -y; \
    yum install -y bash wget tar yum-utils openssh-server net-tools vim \
    device-mapper-persistent-data lvm2; \
    yum update -y ca-certificates --disablerepo=epel; \
    yum clean -y all;
#
#
# ---------  Install Java-JDK  ---------
#
ARG JAVA_VERSION=8
ARG JAVA_RELEASE=JDK
RUN set -eux; \
    pkg="java-1.$JAVA_VERSION.0-openjdk"; \
    if [ "$JAVA_RELEASE" = "JDK" ]; then \
        pkg="$pkg-devel"; \
    else \
        pkg="$pkg-headless"; \
    fi; \
    yum install -y "$pkg"; \
    yum clean all;
ENV JAVA_HOME /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.252.b09-2.el7_8.x86_64/jre/
#
#
# ---------  Install Docker  ---------
#
RUN bash; \
    yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo; \
    yum install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm; \
    yum install -y docker-ce docker-ce-cli;
RUN systemctl enable docker.service;
#
#
# ---------  Install Docker-compose  ---------
#
RUN curl -L "https://github.com/docker/compose/releases/download/1.25.5/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose; \
    chmod +x /usr/local/bin/docker-compose; \
    cp /usr/local/bin/docker-compose /bin/;
#
#
# ---------  Install Maven  ---------
#
ENV MAVEN_LINK  http://mirrors.gigenet.com/apache/maven/maven-3/3.2.5/binaries/apache-maven-3.2.5-bin.tar.gz
RUN cd /tmp; \
    wget -O maven.tar.gz $MAVEN_LINK; \
    cd /opt; \
    mkdir maven; \
    tar xzf /tmp/maven.tar.gz --strip-components=1 -C maven; \
    ln -s /opt/maven/bin/mvn /usr/local/bin; \
    rm -rf /tmp/*;
ENV M2_HOME /opt/maven
#
#
# ---------  Install Tomcat  ---------
#
ENV TOMCAT_LINK https://downloads.apache.org/tomcat/tomcat-8/v8.5.55/bin/apache-tomcat-8.5.55.tar.gz
RUN wget -O /tmp/tomcat-8.tar.gz $TOMCAT_LINK; \
    cd /usr/local; \
    mkdir tomcat-8; \
    tar xzf /tmp/tomcat-8.tar.gz --strip-components=1 -C tomcat-8; \
    rm -rf tomcat-8/webapps/*; \
    ln -s /usr/local/tomcat-8 /usr/local/tomcat; \
    rm -rf /tmp/*; \
    useradd tomcat; \
    chown -R tomcat:tomcat /usr/local/tomcat-8/; \
    chmod +x /usr/local/tomcat-8/bin/*.sh;
COPY config/tomcat.service /etc/systemd/system/
COPY config/server.xml /usr/local/tomcat-8/conf/
RUN systemctl daemon-reload; \
    systemctl start tomcat;
ENV CATALINA_HOME /usr/local/tomcat
#
#
# --------  Install Nginx  ---------
#
RUN yum install -y nginx
COPY config/nginx.conf /etc/nginx/
#
#
# --------  Clean chache yum  ---------
#
RUN rm -rf /var/cache/yum
#
#
# --------  Command run container ---------
#
CMD ["nginx","-g", "daemon off;"]


