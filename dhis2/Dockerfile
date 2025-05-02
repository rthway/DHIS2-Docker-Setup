FROM tomcat:9-jdk11

ENV DHIS2_HOME=/opt/dhis/config

# Create config directory
RUN mkdir -p $DHIS2_HOME

# Copy dhis.conf into the container
COPY dhis.conf $DHIS2_HOME/dhis.conf

# Download and deploy DHIS2 WAR file
RUN apt-get update && apt-get install -y wget && \
    wget -O /usr/local/tomcat/webapps/ROOT.war https://s3-eu-west-1.amazonaws.com/releases.dhis2.org/2.40/dhis.war && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Set JVM memory and DHIS2 config path
ENV JAVA_OPTS="-Xms4000m -Xmx7000m -Ddhis2.home=$DHIS2_HOME"

EXPOSE 8080

CMD ["catalina.sh", "run"]