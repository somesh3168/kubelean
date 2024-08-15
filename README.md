# kubelean

ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 0a:96:35:e5:8c:c9 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 172.31.43.231/20 metric 100 brd 172.31.47.255 scope global dynamic ens5
       valid_lft 2566sec preferred_lft 2566sec
    inet6 fe80::896:35ff:fee5:8cc9/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:ff:ba:a1:e9 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever


       /usr/lib/systemd/system/
sudo nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
helplink[10-kubeadm.conf located under different folder](https://github.com/kubernetes/kubeadm/issues/1575)
sudo nano /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

helplink[Kubeadm init 1700MB limit bug](https://github.com/kubernetes/kubeadm/issues/2365)
sudo kubeadm init --control-plane-endpoint=master-node --upload-certs
sudo kubeadm init --control-plane-endpoint=master-node --upload-certs --ignore-preflight-errors=Mem


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

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join master-node:6443 --token 1r7t3t.7k8mlqrmvbafo9ta \
        --discovery-token-ca-cert-hash sha256:41aa17d69b0331af62eed3a78d7f373259109be2fb090d255a71ea02b54b6e17 \
        --control-plane --certificate-key 7d490594d1d9ebbf7cd343bc45f0d42c32aebe4c068a0fdfe8410274242efed5

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join master-node:6443 --token 1r7t3t.7k8mlqrmvbafo9ta \
        --discovery-token-ca-cert-hash sha256:41aa17d69b0331af62eed3a78d7f373259109be2fb090d255a71ea02b54b6e17 