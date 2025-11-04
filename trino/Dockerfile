FROM trinodb/trino:400

WORKDIR /usr/lib/trino/plugin/hive

RUN wget -O hudi-hadoop-mr-bundle-0.15.0.jar \
    https://repo1.maven.org/maven2/org/apache/hudi/hudi-hadoop-mr-bundle/0.15.0/hudi-hadoop-mr-bundle-0.15.0.jar

COPY ./etc /etc/trino