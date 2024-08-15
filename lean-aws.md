
helpvid [Deploy Kubernetes From Scratch On AWS In 5 Min! (Plus Intro To Kubernetes)](https://youtu.be/vpEDUmt_WKA)

do nthing & [read onn](https://medium.com/@mehmetodabashi/installing-kubernetes-on-ubuntu-20-04-e49c43c63d0c)

Also & [cool enough](https://phoenixnap.com/kb/install-kubernetes-on-ubuntu)


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
    - Tag each instance appropriately, e.g., `master`, `worker-1`, etc., to avoid confusion.

### Step 2: Configure Instances for Kubernetes

1. **SSH into Instances**:
```
chmod 400 your-key.pem
master
ssh -i your-key.pem ubuntu@<Instance_Public_IP>

worker-1
 
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
6. **enable bridging via iptables, do 1 by 1**:
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

### Step 3: Initialize the Control Plane Node

1. **Initialize Kubernetes**:
    ```
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16    aws

    ```
    - The `--pod-network-cidr` is used for the network plugin. This can be adjusted based on your choice of network plugin.
    - use >>> `sudo kubeadm init --pod-network-cidr=127.0.0.1/8 --ignore-preflight-errors=Mem` in Kubeadm init 1700MB limit bug scenario.
    - use >>> `sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors=Mem` in Kubeadm init 1700MB limit bug scenario.

    - reser `sudo kubeadm reset`
      - helplink [Kubeadm init 1700MB limit bug](https://github.com/kubernetes/kubeadm/issues/2365)
      - sudo kubeadm init --control-plane-endpoint=master-node --upload-certs
      - sudo kubeadm init --control-plane-endpoint=master-node --upload-certs --ignore-preflight-errors=Mem
    - copy / save the `sudo kubeadm join <control_plane_ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>` for adding to worker nodes
    
    ```
    Your Kubernetes control-plane has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

    Alternatively, if you are the root user, you can run:

      export KUBECONFIG=/etc/kubernetes/admin.conf

    You should now deploy a pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
      https://kubernetes.io/docs/concepts/cluster-administration/addons/

    Then you can join any number of worker nodes by running the following on each as root:


    kubeadm join masterip --token <token> \
        --discovery-token-ca-cert-hash sha256:<>>
    ```



2. **Set Up kubectl for the ubuntu User**:
    ```
    mkdir -p $HOME/.kube

    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

    sudo chown $(id -u):$(id -g) $HOME/.kube/config

    ```

3. **On the Master server only, apply a common networking plugin. In this case, Flannel**:
    ```
    [deprecated] kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
    kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
    ```
    - Getting somethig like
      - namespace/kube-flannel created
      - clusterrole.rbac.authorization.k8s.io/flannel created
      - clusterrolebinding.rbac.authorization.k8s.io/flannel created
      - serviceaccount/flannel created
      - configmap/kube-flannel-cfg created
    - check for nodes `kubectl get nodes`
    - put it one watch and add workers code `watch kubectl get nodes`

4. **Join Worker Nodes**:
    - On each worker node, run the join command provided at the end of the `kubeadm init` output `sudo` is must:
    - master_ip from nodes `kubectl get nodes` without the `ip-`
    ```
    sudo kubeadm join <master_ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
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
 or 
 2. **Deploy the Nginx Deployment with 2 Pods**
      - Create an Nginx deployment with 2 replicas:
      
      - `kubectl create deployment nginx --image=nginx`
      - `kubectl scale deployment nginx --replicas=2`
 or
 3. **Use raw file(testing)**
    - apply config deployment
    - https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/nginx-deployment.yaml
    - use  `kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/controllers/nginx-deployment.yaml`


        

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
