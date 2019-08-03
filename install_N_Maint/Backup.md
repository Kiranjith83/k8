#  Backing Up and Restoring a Kubernetes Cluster:
  > Backup a cluster comes to one thing its ETCD. 
  > Taking backup of etcd using etcdctl client will backup the cluster state.  
  > Along with the snapshot make sure the /etc/kubernetes/pki/etcd condents are backed-up. This is required at the restore. 
  > Restore option creates an entirely new Cluster, with a different cluster ID. Restore operation overrides the member id / cluster id etc. 
  > New server that needs to be created for replacing an instance in Master cluster, make sure that the server gets the same IP address for successful restore. 
  > Ref: 
  [Backing up cluster ETCD](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)
  [Restore Kubernetes](https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md)
    
###    Steps to take backup:

 Backing up your cluster can be a useful exercise, especially if you have a single etcd cluster, as all the cluster state is stored there. The etcdctl utility allows us to easily create a snapshot of our cluster state (etcd) and save this to an external location. Below steps goes through creating the snapshot.

##### Get the etcd binaries
```
    # wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz
```
#####    Unzip the compressed binaries:
```
    # tar xvf etcd-v3.3.12-linux-amd64.tar.gz
```
#####    Move the files into /usr/local/bin:
```
    # sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin
```
#####    Take a snapshot of the etcd datastore using etcdctl:
```
    # sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key
```
#####    View the help page for etcdctl:
```
    # ETCDCTL_API=3 etcdctl --help
```
#####    Browse to the folder that contains the certificate files:
```
    # cd /etc/kubernetes/pki/etcd/
```
#####    View that the snapshot was successful:
```
    # ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db
```
#####    Zip up the contents of the etcd directory:
```
    # sudo tar -zcvf etcd.tar.gz etcd
```
#####    Copy the etcd directory to another server:
```
        # scp etcd.tar.gz user@ip.my.server:~/
```
