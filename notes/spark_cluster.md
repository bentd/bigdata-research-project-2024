Installing Apache Spark on multiple servers for distributed analytics involves several steps to set up a cluster. Below is a step-by-step guide to achieve this:

### Prerequisites

1. **Multiple Servers**: Ensure you have access to multiple servers. These can be physical servers, virtual machines, or cloud instances.
2. **Java**: Install Java on all nodes.
3. **SSH Access**: Set up passwordless SSH access between the master node and all worker nodes.

### Step 1: Install Java

Install Java on all nodes. Spark requires Java 8 or later.

```sh
sudo apt-get update
sudo apt-get install openjdk-8-jdk -y
```

### Step 2: Download and Install Spark

Download and extract Spark on all nodes (master and worker nodes).

```sh
wget https://downloads.apache.org/spark/spark-3.3.1/spark-3.3.1-bin-hadoop3.tgz
tar -xvf spark-3.3.1-bin-hadoop3.tgz
sudo mv spark-3.3.1-bin-hadoop3 /opt/spark
```

### Step 3: Configure Environment Variables

Add Spark to the PATH in `~/.bashrc` or `~/.profile` on all nodes.

```sh
echo "export SPARK_HOME=/opt/spark" >> ~/.bashrc
echo "export PATH=\$PATH:\$SPARK_HOME/bin:\$SPARK_HOME/sbin" >> ~/.bashrc
source ~/.bashrc
```

### Step 4: Configure Spark

Configure Spark on the master node. Edit the `spark-env.sh` file.

```sh
cp $SPARK_HOME/conf/spark-env.sh.template $SPARK_HOME/conf/spark-env.sh
```

Edit `spark-env.sh` and add the following lines:

```sh
export SPARK_MASTER_HOST='master-node-hostname'
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Ensure that the `SPARK_MASTER_HOST` is set to the hostname or IP address of the master node.

### Step 5: Configure Worker Nodes

On the master node, edit the `slaves` file (also known as `workers` in some Spark versions) to list the worker nodes.

```sh
nano $SPARK_HOME/conf/slaves
```

Add the hostnames or IP addresses of the worker nodes, one per line:

```
worker-node1-hostname
worker-node2-hostname
worker-node3-hostname
```

### Step 6: Start Spark Cluster

#### Start the Master Node

On the master node, start the master service:

```sh
$SPARK_HOME/sbin/start-master.sh
```

#### Start the Worker Nodes

On each worker node, start the worker service. This can be done from the master node if you have SSH access set up:

```sh
$SPARK_HOME/sbin/start-slaves.sh
```

Or you can start each worker manually by SSHing into each worker node and running:

```sh
$SPARK_HOME/sbin/start-slave.sh spark://master-node-hostname:7077
```

### Step 7: Verify the Cluster

Access the Spark Web UI to verify that the cluster is running. Open a web browser and go to:

```
http://master-node-hostname:8080
```

You should see the master node and all the connected worker nodes.

### Step 8: Run a Spark Job

You can now run a Spark job to test the cluster. Here’s an example using the Spark shell:

```sh
$SPARK_HOME/bin/spark-shell --master spark://master-node-hostname:7077
```

In the Spark shell, run a simple job:

```scala
val data = sc.parallelize(1 to 1000, 10)
val result = data.reduce(_ + _)
println(result)
```

### Summary

1. **Install Java**: Ensure Java is installed on all nodes.
2. **Download and Install Spark**: Install Spark on all nodes.
3. **Configure Environment Variables**: Set up environment variables.
4. **Configure Spark**: Edit `spark-env.sh` on the master node.
5. **List Worker Nodes**: Edit the `slaves` file to list worker nodes.
6. **Start Spark Cluster**: Start the master and worker nodes.
7. **Verify the Cluster**: Use the Spark Web UI.
8. **Run a Spark Job**: Test the cluster with a Spark job.

This guide provides the foundational steps to set up a distributed Spark cluster across multiple servers. For production environments, consider additional configurations for security, resource management, and high availability.
