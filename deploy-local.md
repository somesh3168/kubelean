# kubelean

To create a Kubernetes cluster on Ubuntu Focal Fossa and deploy a simple 2-pod Nginx deployment, follow these steps:

### 1. **Prepare Your Ubuntu Machine**

Ensure your system is up-to-date:

```bash
sudo apt-get update
sudo apt-get upgrade -y
```

### 2. **Install Docker**

Kubernetes uses Docker as the container runtime, so you need to install it first:

```bash
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce
or
sudo apt  install docker.io
```

Make sure Docker is running:

```bash
sudo systemctl enable docker
sudo systemctl start docker

```

Add your user to the Docker group to manage Docker as a non-root user:

```bash
sudo usermod -aG docker $USER
newgrp docker
```

### 3. **Install Kubernetes Components (kubeadm, kubelet, kubectl)**

Add Kubernetes' official repository:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
```

Install `kubeadm`, `kubelet`, and `kubectl`:

```bash
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

or

bash
Copy code
sudo snap install kubeadm --classic
If you also want to install kubectl and kubelet via snap, you would do:

bash
Copy code
sudo snap install kubectl --classic
sudo snap install kubelet --classic

```

### 4. **Disable Swap**

Kubernetes requires swap to be disabled:

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### 5. **Initialize the Kubernetes Master Node**

Initialize the Kubernetes cluster on the master node:

```bash
sudo systemctl enable kubelet
sudo systemctl status kubelet
sudo systemctl enable kubelet

sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

After initialization, you'll see a message with instructions to set up `kubectl` for your regular user:

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 6. **Deploy a Pod Network (Calico)**

To allow communication between your Kubernetes pods, you need to install a pod network. Here, we’ll use Calico:

```bash
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 7. **Deploy the Nginx Deployment with 2 Pods**

Create an Nginx deployment with 2 replicas:

```bash
kubectl create deployment nginx --image=nginx
kubectl scale deployment nginx --replicas=2
```

Check if the pods are running:

```bash
kubectl get pods
```

### 8. **Expose the Nginx Deployment**

To expose your Nginx deployment, create a service:

```bash
kubectl expose deployment nginx --port=80 --type=NodePort
```

To get the URL to access the Nginx service, use:

```bash
kubectl get svc
```

### 9. **Access the Nginx Service**

The service should be accessible via `http://<node-ip>:<node-port>`. You can get the node IP by running:

```bash
hostname -I
```

### 10. **Clean Up**

To delete the deployment and service, use:

```bash
kubectl delete deployment nginx
kubectl delete service nginx
```

### Notes

- **Pods**: Basic units of deployment in Kubernetes, usually containing a single container.
- **Deployment**: Manages the deployment of replicas of your application (in this case, Nginx).
- **Service**: Exposes your deployment to the network, making it accessible.

This setup is perfect for a small-scale, learning-focused Kubernetes cluster.