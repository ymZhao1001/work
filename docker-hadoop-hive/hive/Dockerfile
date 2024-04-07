ARG JDK_IMAGE=openjdk:8
FROM $JDK_IMAGE

# Install necessary dependencies
RUN sed -i 's#http://deb.debian.org#https://mirrors.ustc.edu.cn#g' /etc/apt/sources.list && \
    sed -i 's|security.debian.org/debian-security|mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends sudo netcat net-tools
    #wget -q "https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh" -O /usr/local/bin/wait-for && \
    #chmod +x /usr/local/bin/wait-for
COPY wait-for-it.sh /usr/local/bin/wait-for
RUN chmod +x /usr/local/bin/wait-for

# Create Hive user
ENV HIVE_USER=hiveuser
RUN adduser --gecos "" "$HIVE_USER" && \
    adduser "$HIVE_USER" sudo && echo "$HIVE_USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
USER $HIVE_USER

# Add Hadoop distribution
ARG HADOOP_VERSION="3.3.6"
COPY hadoop/hadoop-3.3.6.tar.gz /tmp/hadoop.tar.gz
#wget "https://dlcdn.apache.org/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz" -O /tmp/hadoop.tar.gz && \
RUN tar -xz -f /tmp/hadoop.tar.gz -C "$HOME/" && \
    sudo rm /tmp/hadoop.tar.gz
ENV HADOOP_HOME=/home/hiveuser/hadoop-3.3.6

# Install Hive
ARG HIVE_VERSION="3.1.3"
COPY hive/apache-hive-3.1.3-bin.tar.gz /tmp/hive.tar.gz
#RUN wget "https://dlcdn.apache.org/hive/hive-$HIVE_VERSION/apache-hive-$HIVE_VERSION-bin.tar.gz" -O /tmp/hive.tar.gz && \
RUN tar -xz -f /tmp/hive.tar.gz -C "$HOME/" && \
    sudo rm /tmp/hive.tar.gz
ENV HIVE_HOME=/home/$HIVE_USER/apache-hive-3.1.3-bin
ENV HIVE_CONF_DIR=$HIVE_HOME/conf \
    PATH=$PATH:$HIVE_HOME/bin
ARG AUTHENTICATION_TYPE=kerberos
COPY hive/conf/$AUTHENTICATION_TYPE/ \
     hadoop/conf/$AUTHENTICATION_TYPE/ $HIVE_CONF_DIR/
RUN sudo echo "HADOOP_HOME=$HADOOP_HOME" >> $HIVE_CONF_DIR/hive-env.sh

# Add JDBC driver
#ARG JDBC_DRIVER_JAR="https://repo1.maven.org/maven2/mysql/mysql-connector-java/8.0.27/mysql-connector-java-8.0.27.jar"
#RUN wget $JDBC_DRIVER_JAR -P $HIVE_HOME/lib
COPY hive/mysql-connector-java-8.0.27.jar $HIVE_HOME/lib
# Fix https://issues.apache.org/jira/browse/HIVE-22915
RUN rm -f $HIVE_HOME/lib/guava-19.0.jar && \
    ln -s $HADOOP_HOME/share/hadoop/hdfs/lib/guava-27.0-jre.jar $HIVE_HOME/lib/guava-27.0-jre.jar

ENV AUTHENTICATION_TYPE=$AUTHENTICATION_TYPE
# Kerberos configurations
RUN if [ "$AUTHENTICATION_TYPE" = "kerberos" ]; then \
      # Install Kerberos client
      sudo -E apt-get update && \
      export DEBIAN_FRONTEND=noninteractive && sudo -E apt-get install -y --no-install-recommends krb5-user; \
    fi
COPY kerberos/conf/krb5.conf /etc/krb5.conf

COPY AirlinesTest.csv.zip /tmp/AirlinesTest.csv.zip

# Add entrypoint
COPY hive/entrypoint.sh /tmp/entrypoint.sh
RUN sudo chmod +x /tmp/entrypoint.sh
ENTRYPOINT ["/tmp/entrypoint.sh"]

EXPOSE 9083 10000 10002 55555
