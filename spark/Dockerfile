FROM apache/spark:3.5.6-scala2.12-java11-python3-ubuntu

USER root

ARG UID=1000
ARG GID=1000

# Match container user to host user
RUN groupmod -g ${GID} spark && \
    usermod -u ${UID} spark || true

WORKDIR /opt/spark/

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        openjdk-17-jdk \
        wget \
        procps \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Args for versions
ARG HADOOP_AWS_VERSION=3.3.4
ARG AWS_JAVA_SDK_VERSION=1.11.901
ARG HUDI_VERSION=1.0.2

# Download Jars
RUN mkdir -p /opt/spark/jars && \
    wget -P /opt/spark/jars https://repo1.maven.org/maven2/org/apache/hadoop/hadoop-aws/${HADOOP_AWS_VERSION}/hadoop-aws-${HADOOP_AWS_VERSION}.jar && \
    wget -P /opt/spark/jars https://repo1.maven.org/maven2/com/amazonaws/aws-java-sdk-bundle/${AWS_JAVA_SDK_VERSION}/aws-java-sdk-bundle-${AWS_JAVA_SDK_VERSION}.jar && \
    wget -P /opt/spark/jars https://repo1.maven.org/maven2/org/apache/hudi/hudi-spark3.5-bundle_2.12/${HUDI_VERSION}/hudi-spark3.5-bundle_2.12-${HUDI_VERSION}.jar && \
    apt-get remove -y wget && apt-get autoremove -y

# Copy config
COPY ./conf/ /opt/spark/conf/

# Copy requirements
COPY requirements.txt /tmp/requirements.txt

# Install Python + Jupyter
RUN pip install --no-cache-dir \
        "notebook<7.0" \
        "jupyterlab<4.0" \
        ipykernel \
    && pip install --no-cache-dir -r /tmp/requirements.txt

# Add Jupyter kernel globally (NOT --user!)
RUN python3 -m ipykernel install --name spark-python --display-name "Spark Py3"

# Environment variables
ENV JUPYTER_ENABLE_LAB=yes \
    SPARK_HOME=/opt/spark \
    PYSPARK_PYTHON=python3 \
    PYSPARK_DRIVER_PYTHON=jupyter-lab

# Create workspace & set permissions
RUN mkdir -p /home/spark/work && \
    chown -R spark:spark /home/spark && \
    chown -R spark:spark /opt/spark

WORKDIR /home/spark/work
USER spark

EXPOSE 8888

CMD ["bash", "-c", "jupyter lab --ip=0.0.0.0 --port=8888 --no-browser --NotebookApp.token=\"$JUPYTER_TOKEN\" --NotebookApp.password=\"$JUPYTER_PASSWORD\""]
