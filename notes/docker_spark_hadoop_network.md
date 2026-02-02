Running Spark on a Docker cluster of Hadoop nodes involves several steps, including configuring Spark to use the Hadoop cluster for resource management and data storage. Below is a detailed guide on how to set this up:

### Prerequisites

1. **Hadoop Cluster**: A running Hadoop cluster using Docker, with HDFS and YARN.
2. **Spark Docker Image**: A Docker image for Spark that can interact with the Hadoop cluster.

### Step 1: Ensure Hadoop Cluster is Running

Make sure your Hadoop cluster (NameNode, DataNodes, ResourceManager, NodeManagers) is up and running. This typically involves a `docker-compose.yml` file that defines these services.

Here’s an example `docker-compose.yml` for a basic Hadoop cluster:

```yaml
version: '3.8'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env
    ports:
      - "9870:9870"
      - "9000:9000"
    volumes:
      - namenode-data:/hadoop/dfs/name
    networks:
      - hadoop

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    environment:
      - SERVICE_PRECONDITION=namenode:9870
    env_file:
      - ./hadoop.env
    volumes:
      - datanode-data:/hadoop/dfs/data
    networks:
      - hadoop

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env
    ports:
      - "8088:8088"
    networks:
      - hadoop

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    environment:
      - SERVICE_PRECONDITION=resourcemanager:8088
    env_file:
      - ./hadoop.env
    networks:
      - hadoop

  historyserver:
    image: bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8
    container_name: historyserver
    environment:
      - SERVICE_PRECONDITION=namenode:9870
    env_file:
      - ./hadoop.env
    ports:
      - "8188:8188"
    networks:
      - hadoop

volumes:
  namenode-data:
  datanode-data:

networks:
  hadoop:
    driver: bridge
```

### Step 2: Configure Spark Docker Image

You need a Spark Docker image that can interact with your Hadoop cluster. You can use an existing Spark image or build your own.

Here's an example Dockerfile for Spark:

```Dockerfile
FROM bde2020/spark-base:3.1.1-hadoop3.2

# Install any additional dependencies or configurations
# COPY hadoop-env.sh /etc/hadoop/hadoop-env.sh

ENV SPARK_MASTER_PORT 7077
ENV SPARK_MASTER_WEBUI_PORT 8080
ENV SPARK_WORKER_WEBUI_PORT 8081
ENV HADOOP_CONF_DIR /etc/hadoop

CMD ["/bin/bash"]
```

### Step 3: Docker-Compose for Spark and Hadoop

Extend your `docker-compose.yml` to include Spark Master and Worker services.

```yaml
version: '3.8'

services:
  namenode:
    image: bde2020/hadoop-namenode:2.0.0-hadoop3.2.1-java8
    container_name: namenode
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env
    ports:
      - "9870:9870"
      - "9000:9000"
    volumes:
      - namenode-data:/hadoop/dfs/name
    networks:
      - hadoop

  datanode:
    image: bde2020/hadoop-datanode:2.0.0-hadoop3.2.1-java8
    environment:
      - SERVICE_PRECONDITION=namenode:9870
    env_file:
      - ./hadoop.env
    volumes:
      - datanode-data:/hadoop/dfs/data
    networks:
      - hadoop

  resourcemanager:
    image: bde2020/hadoop-resourcemanager:2.0.0-hadoop3.2.1-java8
    container_name: resourcemanager
    environment:
      - CLUSTER_NAME=test
    env_file:
      - ./hadoop.env
    ports:
      - "8088:8088"
    networks:
      - hadoop

  nodemanager:
    image: bde2020/hadoop-nodemanager:2.0.0-hadoop3.2.1-java8
    container_name: nodemanager
    environment:
      - SERVICE_PRECONDITION=resourcemanager:8088
    env_file:
      - ./hadoop.env
    networks:
      - hadoop

  historyserver:
    image: bde2020/hadoop-historyserver:2.0.0-hadoop3.2.1-java8
    container_name: historyserver
    environment:
      - SERVICE_PRECONDITION=namenode:9870
    env_file:
      - ./hadoop.env
    ports:
      - "8188:8188"
    networks:
      - hadoop

  spark-master:
    image: bde2020/spark-master:3.1.1-hadoop3.2
    container_name: spark-master
    environment:
      - SPARK_MODE=master
      - SPARK_MASTER_PORT=7077
      - SPARK_MASTER_WEBUI_PORT=8080
    ports:
      - "8080:8080"
      - "7077:7077"
    networks:
      - hadoop

  spark-worker:
    image: bde2020/spark-worker:3.1.1-hadoop3.2
    container_name: spark-worker
    environment:
      - SPARK_MODE=worker
      - SPARK_MASTER=spark://spark-master:7077
      - SPARK_WORKER_CORES=2
      - SPARK_WORKER_MEMORY=4g
      - SPARK_WORKER_PORT=8881
      - SPARK_WORKER_WEBUI_PORT=8081
    networks:
      - hadoop
    depends_on:
      - spark-master

volumes:
  namenode-data:
  datanode-data:

networks:
  hadoop:
    driver: bridge
```

### Step 4: Start the Cluster

Run the Docker Compose setup to start the Hadoop and Spark cluster.

```sh
docker-compose up -d
```

### Step 5: Verify the Setup

1. **Hadoop Web UI**: Access the Hadoop NameNode Web UI at `http://<namenode-host>:9870`.
2. **ResourceManager Web UI**: Access the YARN ResourceManager Web UI at `http://<resourcemanager-host>:8088`.
3. **Spark Master Web UI**: Access the Spark Master Web UI at `http://<spark-master-host>:8080`.

### Step 6: Submit Spark Jobs

You can now submit Spark jobs to the cluster. For example, to run a Spark shell:

```sh
docker exec -it spark-master /opt/spark/bin/spark-shell --master spark://spark-master:7077
```

Or to submit a Spark job:

```sh
docker exec -it spark-master /opt/spark/bin/spark-submit --master spark://spark-master:7077 --class org.apache.spark.examples.SparkPi /opt/spark/examples/jars/spark-examples_2.12-3.1.1.jar 10
```

### Summary

1. **Ensure Hadoop Cluster is Running**: Set up a Docker-based Hadoop cluster.
2. **Configure Spark Docker Image**: Use or build a Docker image for Spark.
3. **Extend Docker-Compose**: Add Spark Master and Worker services to `docker-compose.yml`.
4. **Start the Cluster**: Use `docker-compose up -d` to start the cluster.
5. **Verify the Setup**: Access Hadoop and Spark Web UIs.
6. **Submit Spark Jobs**: Use `spark-shell` or `spark-submit` to run jobs on the cluster.

This setup allows you to run Spark on a Docker-based Hadoop cluster, leveraging HDFS for storage and YARN for resource management.
