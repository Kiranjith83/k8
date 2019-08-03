# Installing Kuberbetes
  ---------------------

## Kubernetes on Ubuntu By Linux academy:
   -------------------------------------
   
####   On Master and Node

    Get the Docker gpg key:
```
    #  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
    Add the Docker repository:
```    
    #  sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
```
    Get the Kubernetes gpg key:
```
    #  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```
    Add the Kubernetes repository: (command to work, need to remove the intentation)
```
    #  cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
      deb https://apt.kubernetes.io/ kubernetes-xenial main
      EOF

    # echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
```
    Update your packages:
```    
    #  sudo apt-get update
```
    Install Docker, kubelet, kubeadm, and kubectl:
```    
    #  sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00
```
    Hold them at the current version:
```    
    #  sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```
    Add the iptables rule to sysctl.conf:
```    
    # echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```
    Apply changes immediately:
```    
    #  sudo sysctl -p
```
### Initialize the cluster (run only on the master):
```
    # sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
    Set up local kubeconfig:
```    
    # mkdir -p $HOME/.kube
    # sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    #  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```   
    Apply Flannel CNI network overlay:
```   
    # kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```
###  Join the worker nodes to the cluster(At nodes):
```
    #  kubeadm join [your unique string from the kubeadm init command]
```   
   Example:
```
    #  kubeadm join 172.31.4.246:6443 --token aktv22.266l663emcbmxlqd --discovery-token-ca-cert-hash sha256:20a60cbef61abb1a585360ad95416f11f144e357a4aff34ec579cb305a533030
```   
   Verify the worker nodes have joined the cluster successfully:
```
    #  kubectl get nodes
```
