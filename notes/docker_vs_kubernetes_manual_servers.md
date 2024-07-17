Setting up multiple computers to work as a Docker cluster can be achieved without Kubernetes, but Kubernetes can enhance the management and orchestration capabilities of your Docker cluster. Here’s a breakdown of the options and considerations:

### Option 1: Docker Swarm

**Docker Swarm** is Docker’s native clustering and orchestration tool. It allows you to pool together multiple Docker hosts (machines) into a single virtual system. Here’s how you can set up a Docker Swarm cluster:

1. **Install Docker**: Ensure Docker is installed on each machine that will be part of the cluster. This can be done by following Docker’s installation guides for your operating system.

2. **Initialize Swarm on Manager Node**: Choose one of your machines to be the Swarm manager and initialize the Swarm:

   ```sh
   docker swarm init --advertise-addr <MANAGER-IP>
   ```

   Replace `<MANAGER-IP>` with the IP address of your manager node.

3. **Join Worker Nodes**: On each additional machine that you want to join to the Swarm as a worker node, run:

   ```sh
   docker swarm join --token <TOKEN> <MANAGER-IP>:2377
   ```

   Replace `<TOKEN>` with the token generated during `docker swarm init` on the manager node, and `<MANAGER-IP>` with the IP address of your manager node.

4. **Verify Swarm Status**: On the manager node, you can verify the status of nodes in the Swarm:

   ```sh
   docker node ls
   ```

### Option 2: Kubernetes

**Kubernetes** is a powerful container orchestration platform that can manage Docker containers across multiple hosts. It offers additional features such as automatic scaling, self-healing, and service discovery. Setting up Kubernetes typically involves more setup compared to Docker Swarm but provides more advanced orchestration capabilities:

1. **Setup Kubernetes**: Install Kubernetes on your machines. You can choose to set up Kubernetes manually (using kubeadm, for example) or use managed Kubernetes services like Amazon EKS, Google GKE, or Azure AKS.

2. **Deploy Containers**: Once Kubernetes is set up, you can deploy Docker containers (pods) using Kubernetes manifests (YAML files) that define your applications and services.

### Considerations

- **Complexity**: Docker Swarm is easier to set up and manage compared to Kubernetes, especially for smaller-scale deployments. Kubernetes, while more complex, offers more features and scalability options.
  
- **Features**: Kubernetes provides advanced features like rolling updates, load balancing, and more sophisticated networking and storage options out of the box.

- **Scalability**: Both Docker Swarm and Kubernetes are designed to scale horizontally by adding more nodes to the cluster.

### Summary

- **Docker Swarm**: Easier setup and management, suitable for smaller-scale deployments and simpler use cases.
  
- **Kubernetes**: More complex setup but offers advanced orchestration features, suitable for larger-scale deployments and more complex applications.

Deciding between Docker Swarm and Kubernetes depends on your specific needs, scale of deployment, and level of orchestration and management required for your Docker containers across multiple machines.
