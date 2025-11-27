FROM eclipse-temurin:8-jdk-focal

ENV CATALINA_HOME /usr/local/tomcat
ENV PATH $CATALINA_HOME/bin:$PATH
WORKDIR $CATALINA_HOME

RUN apt-get update && apt-get install -y wget && \
    wget https://archive.apache.org/dist/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.tar.gz -O /tmp/tomcat.tar.gz && \
    tar xvfz /tmp/tomcat.tar.gz -C /tmp && \
    cp -Rv /tmp/apache-tomcat-8.5.24/* /usr/local/tomcat/ && \
    rm -rf /tmp/* /usr/local/tomcat/webapps/*

COPY build/libs/*.war /usr/local/tomcat/webapps/ROOT.war

EXPOSE 8080
CMD ["catalina.sh", "run"]