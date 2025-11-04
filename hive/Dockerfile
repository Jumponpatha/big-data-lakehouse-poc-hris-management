FROM apache/hive:3.1.3

ARG HADOOP_AWS_VERSION=3.3.4
ARG AWS_JAVA_SDK_VERSION=1.12.262

USER root
RUN apt-get update && apt-get install -y wget && rm -rf /var/lib/apt/lists/*

RUN wget -P /opt/hive/lib https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_AWS_VERSION}/hadoop-aws-${HADOOP_AWS_VERSION}.jar \
    && wget -P /opt/hive/lib https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_JAVA_SDK_VERSION}/aws-java-sdk-bundle-${AWS_JAVA_SDK_VERSION}.jar

USER hive