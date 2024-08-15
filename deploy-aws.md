Here's a detailed guide on how to create two Kubernetes clusters from scratch on AWS using t3.micro instances, deploy a sample Nginx server (each cluster with 2 pods), and stay within the free usage tier. I'll break this down step by step, including code snippets that you can use in your README documentation.

helpdoc [k8s install aws ec2 ubuntu 22 lts](https://phoenixnap.com/kb/install-kubernetes-on-ubuntu)
### Prerequisites

1. **AWS Account**: Ensure you have an AWS account. 
2. **AWS CLI**: Install the AWS CLI and configure it with your credentials.
3. **kubectl**: Install `kubectl` for interacting with Kubernetes clusters.
4. **kubeadm, kubelet, and kubectl**: These will be used to set up Kubernetes on the EC2 instances.

### Step 1: Set Up AWS EC2 Instances

1. **Launch EC2 Instances**:
    - Open the EC2 dashboard in AWS.
    - Choose "Launch Instance".
    - Select the **Ubuntu Server 20.04 LTS (HVM), SSD Volume Type** AMI.
    - Choose **t3.micro** for the instance type (this is within the free tier).
    - In the "Configure Instance" step, choose the number of instances. You will create 3 instances per cluster:
        - 1 Control Plane
        - 2 Worker Nodes
    - Under "Configure Security Group", allow the following ports:
        - **22** for SSH
        - **6443** for the Kubernetes API Server
        - **10250** for Kubelet communication
        - **30000-32767** for NodePort services
    - Launch the instances using a key pair you have access to.

2. **Tagging**:
    - Tag each instance appropriately, e.g., `Cluster1-Control-Plane`, `Cluster1-Worker-1`, etc., to avoid confusion.

### Step 2: Configure Instances for Kubernetes

1. **SSH into Instances**:
    ```
    chmod 400 your-key.pem

    ssh -i your-key.pem ubuntu@<Instance_Public_IP>
    ```

2. **Install Docker**:
    ```
    sudo apt update
    
    sudo apt install docker.io -y
    
    sudo systemctl enable docker

    sudo systemctl status docker
    
    sudo systemctl start docker
    
    ```

3. **Add Kubernetes Signing Key**:
    ```
    curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt update

    ```
4. **Install Kubernetes Tools**:
    ```
    sudo apt install kubeadm kubelet kubectl

    sudo apt-mark hold kubeadm kubelet kubectl

    kubeadm version

    ```

5. **Disable Swap**:
    ```
    sudo swapoff -a

    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

    ```

### Step 3: Initialize the Control Plane Node

1. **Initialize Kubernetes**:
    ```
    sudo kubeadm init --pod-network-cidr=192.168.0.0/16

    ```
    - The `--pod-network-cidr` is used for the network plugin. This can be adjusted based on your choice of network plugin.

2. **Set Up kubectl for the ubuntu User**:
    ```
    mkdir -p $HOME/.kube

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    ```

3. **Install a Network Plugin (Weave)**:
    ```
    kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    ```

4. **Join Worker Nodes**:
    - On each worker node, run the join command provided at the end of the `kubeadm init` output:
    ```
    sudo kubeadm join <control_plane_ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
    ```

### Step 4: Repeat for the Second Cluster

- Repeat the above steps to set up a second Kubernetes cluster. Use different instance names and control plane IPs to distinguish between clusters.

### Step 5: Deploy Nginx on Each Cluster

1. **Create a Deployment**:
    ```yaml

    
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: nginx
      template:
        metadata:
          labels:
            app: nginx
        spec:
          containers:
          - name: nginx
            image: nginx:1.14.2
            ports:
            - containerPort: 80
    ```

    ```6. Deploy a Pod Network (Calico)
        To allow communication between your Kubernetes pods, you need to install a pod network. Here, we’ll use Calico:

        bash
        Copy code
        kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
        7. Deploy the Nginx Deployment with 2 Pods
        Create an Nginx deployment with 2 replicas:

        bash
        Copy code
        kubectl create deployment nginx --image=nginx
        kubectl scale deployment nginx --replicas=2
```

2. **Apply the Deployment**:
    ```
    kubectl apply -f nginx-deployment.yaml
    ```

3. **Expose the Deployment**:
    ```
    kubectl expose deployment nginx-deployment --type=NodePort --port=80
    ```

4. **Get the NodePort and Access Nginx**:
    ```
    kubectl get services
    ```
    - Use the public IP of any worker node and the NodePort to access the Nginx service.
5. **Get status check**:
    ```
    sudo journalctl -u kubelet -f
    ```
    - Use the public IP of any worker node and the NodePort to access the Nginx service.

### Step 6: Verify Setup

- Verify that both clusters are running independently and that you can access the Nginx service on both.

### Notes:

- **Costs**: Monitor your AWS usage to ensure you're within the free tier.
- **Clean Up**: Remember to terminate your EC2 instances after your practice to avoid charges.

This guide should give you a comprehensive understanding of setting up Kubernetes clusters from scratch on AWS, deploying applications, and documenting the process.

Let's break down each of these components:

### 1. `kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml`

This command is used to install **Calico**, a popular network plugin for Kubernetes. The `kubectl apply` command fetches the YAML manifest from the specified URL and applies it to your Kubernetes cluster. The YAML manifest contains the necessary configuration to deploy Calico as the networking solution within your cluster.

#### **Why Use Calico?**
- **Networking and Security**: Calico provides networking for pods in Kubernetes clusters and includes powerful network security features like network policies.
- **Performance**: Calico is known for its high performance and scalability, making it a preferred choice in many production environments.
- **Flexibility**: Calico supports multiple data planes, including pure Linux eBPF, IP, and VXLAN.

### 2. Why Use a Network Plugin (Weave)?

A network plugin in Kubernetes is responsible for managing the network connectivity between pods across different nodes in the cluster. **Weave** is one such network plugin.

#### **Why Use Weave?**
- **Simplicity**: Weave is easy to install and use, making it a good choice for learning and development environments.
- **Automatic Peering**: Weave automatically peers all nodes in the cluster, which simplifies configuration.
- **Network Policy Support**: It supports network policies to control the communication between pods.
- **CIDR Flexibility**: Weave allows for flexibility in pod IP addressing without the need for additional configuration.

### 3. `sudo kubeadm init --pod-network-cidr=192.168.0.0/16`

This command initializes the Kubernetes control plane on the master node, with specific network settings.

#### **Explanation of `--pod-network-cidr=192.168.0.0/16`:**
- **Pod Network CIDR**: The `--pod-network-cidr` flag specifies the range of IP addresses that will be used for the pods within the cluster. The `192.168.0.0/16` indicates a block of IP addresses, starting from `192.168.0.0` to `192.168.255.255`, which can be used by pods.
- **Purpose**: This setting is necessary to define a range of IP addresses that will be allocated to pods in the cluster. Different network plugins require different CIDR ranges, so it’s important to match this with the plugin you plan to use.
  
#### **Is It the Same for All AWS EC2 Instances?**
- The `--pod-network-cidr=192.168.0.0/16` value should be the same across all instances in a single cluster because it defines the network range for the entire Kubernetes cluster, not just a single instance.
- However, if you're setting up two separate Kubernetes clusters, you might want to use different `--pod-network-cidr` values to avoid any potential conflicts, especially if the clusters might ever need to communicate or if you're using a shared network.

### **Summary**

- **Calico** is a network plugin focused on high performance and security.
- **Weave** is a simpler, more flexible network plugin, ideal for learning environments.
- The `--pod-network-cidr` sets the IP address range for pods within the Kubernetes cluster, and it must be consistent across all nodes in the cluster.


Certainly! Let's start by examining the content of the `kube-flannel.yml` file, followed by a brief comparison between Flannel and the other network plugin options mentioned earlier (Calico and Weave).

### **Content of `kube-flannel.yml`**

The `kube-flannel.yml` file is a YAML configuration file that defines the necessary components and configurations to deploy Flannel in a Kubernetes cluster. Below is a snapshot of the content within the file:


### **Components in the YAML File**

- **PodSecurityPolicy**: Restricts the privileges of the pods running in Flannel.
- **ServiceAccount**: Ensures that Flannel runs with the appropriate permissions within the cluster.
- **ClusterRole & ClusterRoleBinding**: Provides Flannel with the necessary permissions to operate within the Kubernetes cluster.
- **ConfigMap**: Contains configuration data, including the network configuration (`net-conf.json`) that sets up the overlay network and the CNI plugin configuration (`cni-conf.json`).
- **DaemonSet**: Ensures that the Flannel daemon runs on every node in the cluster, providing consistent networking across the cluster.

### **Comparison: Flannel vs. Calico vs. Weave**

1. **Flannel**:
   - **Type**: Simple overlay network.
   - **Use Case**: Ideal for basic networking needs and small to medium-sized clusters. Suitable for learning environments.
   - **Pros**: 
     - Easy to set up.
     - Minimal configuration.
   - **Cons**: 
     - Limited advanced features like network policies.
     - Performance might not be ideal for large, production environments.
   - **Typical Backend**: VXLAN, host-gw, UDP.

2. **Calico**:
   - **Type**: Pure L3 networking with support for network policies.
   - **Use Case**: Suited for more advanced use cases, including security-focused environments, and can be used in large-scale production clusters.
   - **Pros**: 
     - High performance.
     - Support for advanced network policies.
     - Scalable to large clusters.
   - **Cons**: 
     - More complex setup.
     - Requires more resources.
   - **Backend**: Pure IP networking, BGP, eBPF.

3. **Weave**:
   - **Type**: Overlay network with automatic mesh networking.
   - **Use Case**: Good for clusters with complex topologies or where ease of use is prioritized.
   - **Pros**: 
     - Simple installation and automatic peering.
     - Flexible CIDR handling.
   - **Cons**: 
     - Slightly less performance than Calico.
     - Network policies are not as extensive as in Calico.
   - **Backend**: Fast datapath, VXLAN.

### **Summary**

- **Flannel** is a lightweight and straightforward choice for basic networking needs, particularly in learning or development environments.
- **Calico** offers advanced features and is designed for high-performance production environments with complex networking and security requirements.
- **Weave** is a good middle-ground, balancing simplicity and functionality, making it a solid choice for many scenarios, especially when ease of use is a priority.