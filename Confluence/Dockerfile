FROM ubuntu:trusty
MAINTAINER TBD

# add `Oracle Java JRE` to repository
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys EEA14886 \
    && echo "deb http://ppa.launchpad.net/webupd8team/java/ubuntu $(lsb_release -cs) main" \
       > /etc/apt/sources.list.d/launchpad-webupd8team-java.list \
    && echo debconf shared/accepted-oracle-license-v1-1 select true \
       | sudo debconf-set-selections \
    && echo debconf shared/accepted-oracle-license-v1-1 seen true \
       | sudo debconf-set-selections

# install `Wget`, `Apache Portable Runtime` and `Java 7 JRE` which is supported by `Atlassian Jira`
RUN apt-get update -qq \
   && apt-get install -qqy wget curl net-tools less libtcnative-1 oracle-java7-installer mlocate xmlstarlet \
   && apt-get clean

# Configure timezone and locale
RUN echo "Europe/Berlin" > /etc/timezone && \
    dpkg-reconfigure -f noninteractive tzdata

# Setting Locale 
RUN export LANGUAGE=en_US.UTF-8 && \
    export LANG=en_US.UTF-8 && \
    locale-gen en_US.UTF-8

# setup useful environment variables
ENV JAVA_HOME /usr/lib/jvm/java-7-oracle
ENV CONF_HOME /opt/atlassian/confluence-data
ENV CONF_INSTALL /opt/atlassian/confluence
ENV CONF_VERSION  5.6.6
ENV PATH /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin

# create non-root user to run `Atlassian Confluence`
RUN groupadd confluence
RUN useradd --create-home --comment "Account for running Atlassian Confluence" -g confluence -u 2000 confluence \
    && groupmod -g 2001 confluence  \
    && chmod -R a+rw ~confluence

# install ``Atlassian Confluence``
RUN set -x \
    && mkdir -p /opt/atlassian \
    && chown -R confluence:confluence /opt/atlassian \
    && mkdir -p "${CONF_HOME}" \
    && chown -R confluence:confluence "${CONF_HOME}" \
    && mkdir -p "${CONF_INSTALL}" \
    && chown -R confluence:confluence "${CONF_INSTALL}" \
    && curl -Ls "http://www.atlassian.com/software/confluence/downloads/binary/atlassian-confluence-${CONF_VERSION}.tar.gz" | tar -xz --directory "${CONF_INSTALL}/" --strip-components=1 \
    && chown -R confluence:confluence "${CONF_INSTALL}" \
    && chown -R confluence:confluence "${CONF_HOME}" \
    && chmod -R 775 "${CONF_INSTALL}/temp" \
    && chown -R confluence:confluence "${CONF_INSTALL}/temp" \
    && chmod -R 775 "${CONF_INSTALL}/logs" \
    && chown -R confluence:confluence "${CONF_INSTALL}/logs" \
    && chmod -R 775 "${CONF_INSTALL}/work" \
    && chown -R confluence:confluence "${CONF_INSTALL}/work" \
    && mkdir -p "${CONF_INSTALL}/conf/Standalone" \
    && chmod -R 775 "${CONF_INSTALL}/conf/Standalone" \
    && chown -R confluence:confluence "${CONF_INSTALL}/conf/Standalone" \
    && echo -e  "\nconfluence.home=$CONF_HOME" >> "${CONF_INSTALL}/confluence/WEB-INF/classes/confluence-init.properties" \
    && xmlstarlet ed --inplace \
        --delete "Server/@debug" \
        --delete "Server/Service/Connector/@debug" \
        --delete "Server/Service/Connector/@minProcessors" \
        --delete "Server/Service/Connector/@maxProcessors" \
        --delete "Server/Service/Connector/@useURIValidationHack" \
        --delete "Server/Service/Engine/@debug" \
        --delete "Server/Service/Engine/Host/@debug" \
        --delete "Server/Service/Engine/Host/Context/@debug" \
                 "${CONF_INSTALL}/conf/server.xml"

#Setting CONTEXT_PATH Variable:-
ENV CONTEXT_PATH /confluence
RUN mv "${CONF_INSTALL}/conf/server.xml"  "${CONF_INSTALL}/conf/server-backup.xml"
RUN xmlstarlet ed -u '//Context/@path' -v "$CONTEXT_PATH" "${CONF_INSTALL}/conf/server-backup.xml"  > "${CONF_INSTALL}/conf/server.xml"
RUN chmod 775 "${CONF_INSTALL}/conf/server.xml"

#Adding fix for:- Invalid file name for use with -Xloggc: Filename can only contain the characters [A-Z][a-z][0-9]-_.%[p|t] but it has been /opt/atlassian/confluence/logs/gc-2015-01-25_13:38:39.
ADD setenv.sh /opt/atlassian/confluence/bin/setenv.sh
RUN chown -R confluence:confluence /opt/atlassian/confluence/bin/setenv.sh
RUN chmod 775 /opt/atlassian/confluence/bin/setenv.sh
RUN touch /opt/atlassian/confluence/logs/gc-2015-02-10.log
RUN chmod 775 /opt/atlassian/confluence/logs/gc-2015-02-10.log
RUN chown -R confluence:confluence /opt/atlassian/confluence/logs/gc-2015-02-10.log
ADD ojdbc6.jar /opt/atlassian/confluence/confluence/WEB-INF/lib/ojdbc6.jar
RUN chown -R confluence:confluence /opt/atlassian/confluence/confluence/WEB-INF/lib/ojdbc6.jar
RUN chmod 644 /opt/atlassian/confluence/confluence/WEB-INF/lib/ojdbc6.jar

# run ``Atlassian Confluence`` as unprivileged user by default
USER confluence:confluence

# expose default ``Atlassian Confluence`` HTTP port
EXPOSE 8090

# set workingdir:-
WORKDIR /opt/atlassian

# set volume mount points for installation and home directory
VOLUME ["/opt/atlassian", "/opt/atlassian/confluence", "/opt/atlassian/confluence-data"]

# run ``Atlassian Confluence`` as a foreground process by default
ENTRYPOINT ["/opt/atlassian/confluence/bin/start-confluence.sh", "-fg"]
