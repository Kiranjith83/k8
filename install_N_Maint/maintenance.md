
# Upgrading Kubernetes
-------------------------
 kubeadm allows us to upgrade the cluster components in sequence order of the version, making sure to include important feature upgrades we might want to take advantage of in the latest stable version of Kubernertes. We will go through upgrading our cluster from version 1.13.5 to 1.14.1.

### On Master 

##### View the version of the server and client on the master node
```
  # kubectl version --short
```
#####  View the version of the scheduler and controller manager
```
  # kubectl get pods -n kube-system kube-controller-manager-chadcrowell1c.mylabserver.com -o yaml
```
#####  View the name of the kube-controller pod
```
  # kubectl get pods -n kube-system
```
#####  Set the VERSION variable to the latest stable release of Kubernetes
```
  # export VERSION=v1.14.1
```
#####  Set the ARCH variable to the amd64 system
```
  # export ARCH=amd64
```
#####  View the latest stable version of Kubernetes using the variable
```
  # echo $VERSION
```
#####  Curl the latest stable version of Kubernetes
```
  # curl -sSL https//dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubeadm > kubeadm
```
#####  Install the latest version of kubeadm
```
  # sudo install -o root -g root -m 0755 ./kubeadm /usr/bin/kubeadm
```
#####  Check the version of kubeadm
```
  # sudo kubeadm version
```
#####  Plan the upgrade
```
  # sudo kubeadm upgrade plan
```
#####  Apply the upgrade to 1.14.1
```
  # kubeadm upgrade apply v1.14.1
```
#####  View the differences between the old and new manifests
```
  # diff kube-controller-manager.yaml /etc/kubernetes/manifests/kube-controller-manager.yaml
```
#####  Curl the latest version of kubelet
```
  # curl -sSL https//dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubelet > kubelet
```
#####  Install the latest version of kubelet
```
  # sudo install -o root -g root -m 0755 ./kubelet /usr/bin/kubelet
```
#####  Restart the kubelet service
```
  # sudo systemctl restart kubelet.service
```
####  Watch the nodes as they change version
```
  # kubectl get nodes -w
```
### Considerations while Upgrading Node for maintenance
  ------------------------------------------------------
  > If need has to be taken nodes down for maint, have to make sure the pod is moved to a diff node for Availability. 
  > If downtime of a node is more than 5 minutes the pod on node will automatically get terminated by controller. 
  > When we need to take a node down for maintenance, Kubernetes makes it easy to evict the pods on that node, take it down, and then continue scheduling pods after the maintenance is complete. Furthermore, if the node needs to be decommissioned, you can just as easily remove the node and replace it with a new one, joining it to the cluster.
 
####  Process to evict the pods and start maintenance on a node
#####    See which pods are running on which nodes
```
     # kubectl get pods -o wide
```
#####    Evict the pods on a node
```
     # kubectl drain [node_name] --ignore-daemonsets
```
#####    Watch as the node changes status
```
     # kubectl get nodes -w
```
#####    Schedule pods to the node after maintenance is complete
```
     # kubectl uncordon [node_name]
```
#####    Remove a node from the cluster
```
     # kubectl delete node [node_name]
```  
####  Add new node to the exiting cluster:

##### Generate a new token:
```
     # sudo kubeadm token generate
```
##### List the tokens:
```
     # sudo kubeadm token list
```
##### Print the kubeadm join command to join a node to the cluster:
```
     # sudo kubeadm token create [token_name] --ttl 2h --print-join-command
```
