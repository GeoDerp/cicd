FROM registry.access.redhat.com/ubi9/ubi:latest AS base


ARG USER_ID=1001
ARG GROUP_ID=1001
ENV USER_NAME=default

ENV HOME="/app"
ENV PATH="/app/.local/bin:${PATH}"


USER root

# Check for package update
RUN dnf -y update-minimal --security --sec-severity=Important --sec-severity=Critical && \
# Install git, nano & java 
dnf install git nano java-21-openjdk -y; \
# clear cache
dnf clean all

WORKDIR ${HOME}

# Create user and set permissions
RUN groupadd -g ${GROUP_ID} ${USER_NAME} && \
    useradd -u ${USER_ID} -r -g ${USER_NAME} -d ${HOME} -s /bin/bash ${USER_NAME} 

#-----------------------------

# Dev target
FROM base AS dev
COPY .devcontainer/devtools.sh /tmp/devtools.sh
# Install extra dev tools as root, then run as default user
RUN chmod +x /tmp/devtools.sh && /tmp/devtools.sh  
USER ${USER_NAME}

# DEPLOYMENT EXAMPLE:
#-----------------------------

# Prod target
FROM base

## Install tomcat server (EXAMPLE)
ADD https://dlcdn.apache.org/tomcat/tomcat-11/v11.0.5/bin/apache-tomcat-11.0.5.tar.gz apache-tomcat-11.0.5.tar.gz
## REPLACE: replace username and password
RUN tar xvf apache-tomcat-*.tar.gz; \
mv apache-tomcat-11.0.5 /usr/local/tomcat11/; \
<<EOF cat >> /usr/local/tomcat11/conf/tomcat-users.xml
  <tomcat-users>
      <role rolename="manager-gui"/>
      <role rolename="admin-gui"/>
      <user username="${USER_NAME}" password="${USER_NAME}" roles="manager-gui,admin-gui"/>
  </tomcat-users>
EOF

## Move to app folder, copy project into container
WORKDIR ${HOME}

## REPLACE: replace this COPY statement with project specific files/folders
COPY . . 

## Check home and apache folder
RUN chown -R ${USER_NAME}:${USER_NAME} ${HOME} && \
    chmod -R 0750 ${HOME}; \
    chown -R default:default /usr/local/tomcat11/webapps/ 

USER ${USER_NAME}

## Install project requirements, build project
RUN ./gradlew clean war; \
cp ./app/build/libs/*.war /usr/local/tomcat11/webapps/

## Expose port and run app
EXPOSE 8080
CMD [ "/usr/local/tomcat11/bin/startup.sh"  ]