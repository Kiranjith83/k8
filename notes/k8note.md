
# THIS IS DRAFT AND NOT TO BE REFERRED: 
## TYPOS, AND UNTESTED CONCEPTS AHEAD ! 
=======================


Links:
=======================
> Getting Started.
https://www.youtube.com/watch?v=_vHTaIJm9uY


Terms and concepts:
=======================


 kubectl:
  Command line tool to manage the K8s. Administrative tool for Orchestrating and managing the service.


  =======================
   Installing Kubernetes=
  =======================
   Kubernetes on Ubuntu By Linux academy:
   -------------------------------------
   Get the Docker gpg key:
    #  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

   Add the Docker repository:
    #  sudo add-apt-repository    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"

   Get the Kubernetes gpg key:
    #  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

   Add the Kubernetes repository: (command to work, need to remove the intentation)
    #  cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
      deb https://apt.kubernetes.io/ kubernetes-xenial main
      EOF

      # echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list

   Update your packages:
    #  sudo apt-get update

   Install Docker, kubelet, kubeadm, and kubectl:
    #  sudo apt-get install -y docker-ce=18.06.1~ce~3-0~ubuntu kubelet=1.13.5-00 kubeadm=1.13.5-00 kubectl=1.13.5-00

   Hold them at the current version:
    #  sudo apt-mark hold docker-ce kubelet kubeadm kubectl

   Add the iptables rule to sysctl.conf:
    # echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf

   Enable iptables immediately:
    #  sudo sysctl -p

   Initialize the cluster (run only on the master):
    # sudo kubeadm init --pod-network-cidr=10.244.0.0/16

   Set up local kubeconfig:
    # mkdir -p $HOME/.kube
    # sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    #  sudo chown $(id -u):$(id -g) $HOME/.kube/config
   Apply Flannel CNI network overlay:
    # kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

   Join the worker nodes to the cluster:
    #  kubeadm join [your unique string from the kubeadm init command]
    nodes at linux academy
    #  kubeadm join 172.31.4.246:6443 --token aktv22.266l663emcbmxlqd --discovery-token-ca-cert-hash sha256:20a60cbef61abb1a585360ad95416f11f144e357a4aff34ec579cb305a533030
     EC2 instance node 
    # kubeadm join 192.168.236.61:6443 --token 6frvwt.em4vye9gpwj808a0 --discovery-token-ca-cert-hash sha256:518eede3731cf6fe3dde6b9bbdd03adca7f46d79dd4f9f91e6abb832abc560bd
      
   Verify the worker nodes have joined the cluster successfully:
    #  kubectl get nodes

  Securing Cluster Communication:
  ------------------------------
   - Recommended to use TLS communication. 
   - kubectl is a translater for kube API server. 
     > kubectl create pod sends a Rest api server, identifies the user by authentications server and sends to authorization server. Once authorized it sends to adminission control to perform the action. For example, removing the pod. Then information is stored in etcd and gets a response back.
   - By default the authentication is from kubectl is performed using the certificate. 

  Test end to End connection on Kubernetes:
  ----------------------------------------
  > kubetest can be used for end to end testing of the application. 
   Ref: https://github.com/kubernetes/community/blob/master/contributors/devel/sig-testing/e2e-tests.md
  > To test:
   - Create a deployment 
   - check if pods can run. 
   - check if pod can be accessible. 
    Use port forwarding to access a pod directly:
    # kubectl port-forward $pod_name 8081:80
    Get a response from the nginx pod directly:
    # curl --head http://127.0.0.1:8081
   - check if logs can be collected 
   - Check running commands from the pods. 
   - Expose the depoyment and see if the service is running. 
    # kubectl expose deployment nginx --port 80 --type NodePort
    List the services in your cluster:
    # kubectl get services
    Get a response from the service:
    # curl -I localhost:$node_port
   - check the nodes are healthy. 

 =======================
  Maintainance on K8s
 =======================

Upgrading the Kubernetes=:
-------------------------
kubeadm allows us to upgrade our cluster components in the proper order, making sure to include important feature upgrades we might want to take advantage of in the latest stable version of Kubernertes. In this lesson, we will go through upgrading our cluster from version 1.13.5 to 1.14.1.

  View the version of the server and client on the master node:
  # kubectl version --short

  View the version of the scheduler and controller manager:
  # kubectl get pods -n kube-system kube-controller-manager-chadcrowell1c.mylabserver.com -o yaml

  View the name of the kube-controller pod:
  # kubectl get pods -n kube-system

  Set the VERSION variable to the latest stable release of Kubernetes:
  # export VERSION=v1.14.1

  Set the ARCH variable to the amd64 system:
  # export ARCH=amd64

  View the latest stable version of Kubernetes using the variable:
  # echo $VERSION

  Curl the latest stable version of Kubernetes:
  # curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubeadm > kubeadm

  Install the latest version of kubeadm:
  # sudo install -o root -g root -m 0755 ./kubeadm /usr/bin/kubeadm

  Check the version of kubeadm:
  # sudo kubeadm version

  Plan the upgrade:
  # sudo kubeadm upgrade plan

  Apply the upgrade to 1.14.1:
  # kubeadm upgrade apply v1.14.1

  View the differences between the old and new manifests:
  # diff kube-controller-manager.yaml /etc/kubernetes/manifests/kube-controller-manager.yaml

  Curl the latest version of kubelet:
  # curl -sSL https://dl.k8s.io/release/${VERSION}/bin/linux/${ARCH}/kubelet > kubelet

  Install the latest version of kubelet:
  # sudo install -o root -g root -m 0755 ./kubelet /usr/bin/kubelet

  Restart the kubelet service:
  # sudo systemctl restart kubelet.service

  Watch the nodes as they change version:
  # kubectl get nodes -w

  Upgrading Node for maintenance:
  -------------------------------
  > If need has to be taken down for maint, have to make sure the pod is moved to a diff node for Availability. 
  > If downtime of a node is more than 5 minuts the pod on node will automatically get terminated by controller. 
  When we need to take a node down for maintenance, Kubernetes makes it easy to evict the pods on that node, take it down, and then continue scheduling pods after the maintenance is complete. Furthermore, if the node needs to be decommissioned, you can just as easily remove the node and replace it with a new one, joining it to the cluster.
 
  Process to evict the pods and start maintance on a node:
    See which pods are running on which nodes:
     # kubectl get pods -o wide

    Evict the pods on a node:
     # kubectl drain [node_name] --ignore-daemonsets

    Watch as the node changes status:
     # kubectl get nodes -w

    Schedule pods to the node after maintenance is complete:
     # kubectl uncordon [node_name]

    Remove a node from the cluster:
     # kubectl delete node [node_name]
  
   Add new node to the exiting cluster:
   -------------------------------
    Generate a new token:
     # sudo kubeadm token generate

    List the tokens:
     # sudo kubeadm token list

    Print the kubeadm join command to join a node to the cluster:
     # sudo kubeadm token create [token_name] --ttl 2h --print-join-command



  Backing Up and Restoring a Kubernetes Cluster:
  ---------------------------------------------
  > Backup a cluster comes to one thing its ETCD. 
  > Taking backup of etcd using etcdctl client will backup the cluster state.  
  > Along with the snapshot make sure the /etc/kubernetes/pki/etcd condents are backedup. This is required at the restore. 
  > Restore option creates an entirely new Cluster, with a different cluster ID. Restore operation overrides the member id / cluster id etc. 
  > New server that needs to be created for replacing an instance in Master cluster, make sure that the server gets the same IP address for successful restore. 
  Ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
       https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/recovery.md
    
    Steps to take backup:
    --------------------
    Backing up your cluster can be a useful exercise, especially if you have a single etcd cluster, as all the cluster state is stored there. The etcdctl utility allows us to easily create a snapshot of our cluster state (etcd) and save this to an external location. In this lesson, we’ll go through creating the snapshot and talk about restoring in the event of failure.

        Get the etcd binaries:
        # wget https://github.com/etcd-io/etcd/releases/download/v3.3.12/etcd-v3.3.12-linux-amd64.tar.gz

        Unzip the compressed binaries:
        # tar xvf etcd-v3.3.12-linux-amd64.tar.gz

        Move the files into /usr/local/bin:
        # sudo mv etcd-v3.3.12-linux-amd64/etcd* /usr/local/bin

        Take a snapshot of the etcd datastore using etcdctl:
        # sudo ETCDCTL_API=3 etcdctl snapshot save snapshot.db --cacert /etc/kubernetes/pki/etcd/server.crt --cert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/ca.key

        View the help page for etcdctl:
        # ETCDCTL_API=3 etcdctl --help

        Browse to the folder that contains the certificate files:
        # cd /etc/kubernetes/pki/etcd/

        View that the snapshot was successful:
        # ETCDCTL_API=3 etcdctl --write-out=table snapshot status snapshot.db

        Zip up the contents of the etcd directory:
        # sudo tar -zcvf etcd.tar.gz etcd

        Copy the etcd directory to another server:
        # scp etcd.tar.gz cloud_user@18.219.235.42:~/



=======================
 Master components:
=======================
   API Server (kube-apiserver):
     Exposing the set of APIs for all the operations. kubectl binary connects to the API Server. Kubernetes Dashboard also makes use of the same.  kubectl CLI - is the CLI that manages the Master. The restful API call is made to the kube-apiserver.
     Every components communicates with API Server. 

   Scheduler(kube-scheduler):
     Schedules the pods across the nodes. Listens to new pod creation request. 
     Scheduling rules are used to schedule pods to nodes. 
     There are number of checks operformed by Scheduler prior to schedule a pod on node.
      1. Does node have adequate hardware resources. 
      2. Is the node is running out of resource. (reporting memory or disk pressure issues)
      3. Pod request needs to be scheduled to a node by name 
      4. Does the node have a matching label.
      5. If the pod is request a port that is bound to a specific node 
      6. If the pod requests for a specific volume and it can be mounted. 
      7. Does the pod tolerate the taint of the node .
      8. Does the pods specify node or pod affinity rules.  
       - after check the scheduler will have more than 1 node, then it starts scheduling with the priority. If both nodes have same priority it will schedule on round robin fassion. Besides this scheduler choosing nodes on priority it also makes use of selector spread priority function. This is to make sure that the pods in same replicaset is spread accross different nodes.
          View the name of the scheduler pod:
          # kubectl get pods -n kube-system

          Get the information about your scheduler pod events:
          # kubectl describe pods [scheduler_pod_name] -n kube-system

          View the events in your default namespace:
          # kubectl get events

          View the events in your kube-system namespace:
          # kubectl get events -n kube-system

          Delete all the pods in your default namespace:
          # kubectl delete pods --all

          Watch events as they are appearing in real time:
          # kubectl get events -w

          View the logs from the scheduler pod:
          # kubectl logs [kube_scheduler_pod_name] -n kube-system

          Scheduler can run as a daemon in master node. The location of a systemd service scheduler pod:
          # /var/log/kube-scheduler.log

   Controller Manager(kube-controller-manager): 
     Co-orination operation, Health monitor and checks if the desired state is maintained. Runs Controllers. Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process. Controllers include Node Controller, Replication Controller, Endpoints Controller and Service Account & Token Controller. 

   Cloud Controller Manager: 
     Allows Cloud Vendors code and Kube Controller code to evolve independent of each other. Example - The node controller has a cloud provider dependency, in order for the cloud provider to determine if a node has been deleted in the cloud after it stops responding.

   ETCD: 
     Central DB to store the current state of the cluster.Its a Key value DB. Single source of location for all k8 master components to grab the details. 

  # kubectl get componentstatus
    NAME                 STATUS    MESSAGE              ERROR
    controller-manager   Healthy   ok                   
    scheduler            Healthy   ok                   
    etcd-0               Healthy   {"health": "true"}
     - Command tells current status of kubernetes components. 

    Nodes: Will pull the docker images from the container and runs.
    Minikube: Is a single node standalone kubernetes cluster service for getting started. 
    Kubernetes Multi-Node: -> For production, testing advanced scenarios. 

     Running Multiple Scheduler for Multiple pods: 
     --------------------------------------------
     It is possible to create multiple schedulers for the pods.
     Ref: https://kubernetes.io/docs/tasks/administer-cluster/configure-multiple-schedulers/
     in Pod spec, schedulerName: specifies the scheduler for scheduling the pods. 
    Example: 
    The YAML for the new scheduler:
    # scheduler.yaml
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      labels:
        component: scheduler
        tier: control-plane
      name: my-scheduler
      namespace: kube-system
    spec:
      replicas: 1
      template:
        metadata:
          labels: 
            component: scheduler
            tier: control-plane
            version: second
        spec:
          containers:
          - command: [/usr/local/bin/kube-scheduler, --address=0.0.0.0,
                      --scheduler-name=my-scheduler, --leader-elect=false]
            image: linuxacademycontent/content-kube-scheduler
            livenessProbe:
              httpGet:
                path: /healthz
                port: 10251
              initialDelaySeconds: 15
            name: kube-second-scheduler
            readinessProbe:
              httpGet:
                path: /healthz
                port: 10251
            resources:
              requests:
                cpu: '0.1'
            securityContext:
              privileged: false
            volumeMounts: []
          hostNetwork: false
          hostPID: false
          volumes: []

    Run the deployment for my-scheduler:
    # kubectl create -f my-scheduler.yaml

    View your new scheduler in the kube-system namespace:
    # kubectl get pods -n kube-system

    The pod YAML for pod 1:
    # cat pod1.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: no-annotation
      labels:
        name: multischeduler-example
    spec:
      containers:
      - name: pod-with-no-annotation-container
        image: k8s.gcr.io/pause:2.0

    The pod YAML for pod 2:
    # cat pod2.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: annotation-default-scheduler
      labels:
        name: multischeduler-example
    spec:
      schedulerName: default-scheduler
      containers:
      - name: pod-with-default-annotation-container
        image: k8s.gcr.io/pause:2.0

    The pod YAML for pod 3:
    # cat pod3.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: annotation-second-scheduler
      labels:
        name: multischeduler-example
    spec:
      schedulerName: my-scheduler
      containers:
      - name: pod-with-second-annotation-container
        image: k8s.gcr.io/pause:2.0

    View the pods as they are created:
    # kubectl get pods -o wide

 
  ---------------------------
  Managing High Availability:
  ---------------------------
   # kubectl get pods -o custom-columns=POD:metadata.name,NODE:spec.nodeName --sort-by spec.nodeName -n kube-system
   # kubectl get endpoints kube-scheduler -n kube-system -o yaml

    In Kubernetes Controller and scheduler in master will have only one at active at a time. This is controller by leader elect option. Leader elect is performed by creating the endpoint resource. 
   This can be visible using below command:
   # kubectl get endpoints kube-scheduler -n kube-system -o yaml
      apiVersion: v1
      kind: Endpoints
      metadata:
        annotations:
          control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"ip-192-168-236-61_30b5fedc-a39c-11e9-a3e6-0ad57078477c","leaseDurationSeconds":15,"acquireTime":"2019-07-11T05:24:54Z","renewTime":"2019-07-11T05:55:15Z","leaderTransitions":0}'
        creationTimestamp: "2019-07-11T05:24:54Z"
        name: kube-scheduler
        namespace: kube-system
        resourceVersion: "3659"
        selfLink: /api/v1/namespaces/kube-system/endpoints/kube-scheduler
        uid: 3745327f-a39c-11e9-9e89-0ad57078477

    As the annotation says "control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"ip-192-168-236-61_30b5fedc-a39c-11e9-a3e6-0ad57078477c"," is the current leader. 

   Replicating the ETCD:
    Can be replicated for high Availability
     # kubeadm init --config=kubeadm-config.yaml

 Communication From Master to Nodes:
 ----------------------------------
  From Master to cluster Kubelete and Nodes or Pods are on plain text. 


 =======================
  Objects
 =======================
 Management Methods:
 ------------------
 Imperative Commands:
  Example command#
   # kubectl run nginx --image nginx 
     This creates a deployment from the image nginx. 
   # kubectl expose ..
   # kubectl autoscale 
    - Simple, but No audit, review or cannot be versioned. 

 Imperative Object Configuration:
  # Here we define a yaml file and use the file to create the pods. 
  Example:
   # kubectl create -f pod.yaml 
   # kubectl replace -f pod.yaml 
   # kubectl delete -f config.yaml  

 Declarative Object Configuration: 
  # Prefered way of working, provides the final end state that is desired using the yaml/json file.
  Example: 
  # kubectl apply -f config/
  - Most robust way. Can be tracked with version controlling.  
  - K8s will parse all folders and make changes.
  - Complexity. 

  Applying Declarative object configuration:
  ------------------------------------------
  - Live Object configuration:  Current running status.
  - Current Object Configuration: The intend change going to apply. 
  - Last-applied Object Configuration: The last change applied. 
    Control plane decides the change comparing above Object configurations. The change is merged as below:
    > Primitive fields:
     String, int, boolean, images or replicas - replaces old state with current Object configuration. 
    > Map fields:
     Merges old state with current object configuration file. 
    > list field:
     It is comples and varies by field. 

  Object Names and UIDs 
  ---------------------
  Objects are persistent entities
   - pods, replicasets, services, volumes and nodes.
   - The information is saved in ETCD 
   All objects in K8s are identified with 
   - Names 
   - UUIDs 

 =======================
  NameSpaces
 =======================
  - Divding physical cluster into multiple virutal clusters.
  - By default three namespaces are created. 
     - default: Objects are created if none is specified. 
     - kube-system: For internal K8 objects. 
     - kube-public: Auto readable by all users. 
  - Objects only needed to have unique name within a namespace. 
  - Low level objects such as Nodes, PersistentVolumes and NameSpace themsleves are not part of any Namespace. 

 =======================
  Node components: 
 =======================
  A machine that runs the Pods or K8 applications. 
  Kube-proxy:
      It manages the Iptables and network tables. It maintains the distributed network and exposes the services internally or externally. If running multi nodes and a pod is available on a single node, the Kubeproxy runs the IPtables redirection to make sure the traffic reaches the right pod and node when accessing the service using any IP of the nodes. 
      - LoadBalances traffic between application components like service components. 
      - IP address associated with a service is a virtual IP address, meaning no physical nic. Once a service is created it gets a virtual IP from the master, And notifies kube-proxy that the new service has been created. Kube-proxy then makes sure that the service is reachable by creating the IPTables rules. 
      - Once Iptables rules is created, and if a pod is trying to access the service the destination IP and port will be the Service Virtual IP. 
      - Once it reaches IPtables rules, the destination changes to the IP and Port of a dynamically selected pod which is part of the Service.

  
  Container Runtime: 
      Software that responsible for running  containers. K8s supports several runtimes docker, rocker and runc.. most commonly used it docker.  
  Kubelet:
     Is the agent responsible for API and vice versa, reporting health metrics and current state of the pods. Listens to the instructions coming from the master. 
  Supervisord:
    Both docker and kubelet are packaged into a layer called "supervisord"
  Flunetd:
    Manages the logs of pods.
  

  Containers: 
    Are the lowest denomination units. 
  =======================
    kubelete 
  =======================
    EKS Cloud Controller, reads the tags and adds the nodes to EKS Control plane. 
    Logs location: /var/log/messages 
    

  =======================  
  Pods: 
  =======================
    Are the collection of homogeneous containers. (Like your application and Messaging queues containers together). Fundamental unit of deployment.
     Containers in same POD share same IP address and can communicate locally. With 127.0.0.1. A Pod is like a Virtual Machine, and each container inside the pod is like a Process in a system.
      Inside a pod, containers share the same volume as well. As much as possible the containers running should be stateless. Pod share the network, IP address, etc.
    
    Types of Pods: 
    > Single Container Pods:
       Pod with a single container. 

    > Multi Container Pods:
       Pod running with multiple applications. Shares Memory space, Volumes, and can be connected with localhost. Same paramters(ConfigMaps) are shared. If one Container crashes, it kills the entire pods. 

  A pod runs only on a single Node. 

  Containers inside a pod. 
   A pod can have different roles of containes. 
   - init container. -> Always runs before the setup of an applicaiton environment.
    Example: Running setup utility for the app container. 
             Writing config files in volume.
             Waiting for a length of time. 
             Cloning a git repo or pull latest version of some files.
   - Sidecar container. -> fetch data for other containers and connects to other services. 
    Example: Pull message from a queue and forward then to app container
             Use log streaming. 
   - Containers in a pod share same net namespace allowing to access with local hosts. 
   - Containers can share storage, and can be mounted read/write/readonly per conatiner..

   Lifecycle of a POD:
   -------------------
   - kubectl apply -f pod.yaml -> will be picked up by the kube-apiserver. 
   - Parsed and handed over to the kube-scheduler.
   - Scheduler communicates to kubelete running on the nodes to provision the pod
   - If everything goes well, a pod comes up, with containers. 
   - Kubelete send a success/failure message to master.
   - Pods are not resiliant to Nodes failure. Failure has to be managed at the higher level. 
   - PodStatus.Phase tells the status of the POD,    
      Pods Status:
        Pending:
        Kubernetes is pulling images, getting ready to run a pod. 
        Running
        At least one container in the pod is curently runnign, starting or restarting. 
        Succeeded
        All containers STOPPED, all ran into completion without existing in error. 
        Failed
        All containers STOPPED, at least one exited in error or had to be killed.
        Unknown
        Pod status couldn't be obtained.
   - Restart Policy of Containers: 
     - Always (default):
     - On-Failure:
     - Never: 

   - Deploying a Pod:
     Define a Pod in Yaml -> Submitted to the Master -> Parses the definition -> Chooses the node and hands over to node -> Node parses the definition and pulls the image from the repo and provisions the pods and reports the health back to the master. 
   
   Create a Single container POD:
   -----------------------------
   # kubectl run firstdeployment --image=nginx 
   deployment.apps/firstdeployment created
     - Creates a deployment and runs the nginx container in a pod. 
   # kubectl delete deployment/firstdeployment
     - Deletes the deployment created with kubectl run command. 
   # kubectl exec -ti firstdeployment-c54555576-fhx4q -- /bin/bash
     - gets an internactive shell to the container. 

   Create a Multi Container POD: 
   ----------------------------
   # create multipod container yaml file. 
    kind: Pod
    apiVersion: v1
    metadata:
      name: frontend
    spec:
      containers:
      - name: db
        image: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
      - name: wp
        image: wordpress
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"

  # kubectl create -f  resource-limited-pod.yaml 
  pod/frontend created
    - Creation has started 

  # kubectl get pods
  NAME                              READY   STATUS              RESTARTS   AGE
  firstdeployment-c54555576-fhx4q   1/1     Running             0          1h
  frontend                          0/2     ContainerCreating   0          11s
    - Zero out of two containers are in ready state, they are in creation phase. 
  
  # kubectl get pods
  NAME                              READY   STATUS    RESTARTS   AGE
  firstdeployment-c54555576-fhx4q   1/1     Running   0          1h
  frontend                          2/2     Running   0          26s
   - All containers in the pods are up and running. 

  Deleting Pods:
  -------------
  # kubectl delete pods --all
   This will delete all the pods, but pods created with Imperative method, or running by deployment will be restarted. 


  Containers in POD
  -----------------
  > Images should be prebuild and made available on container registry for K8s to work. 
  Ways to pull Images:
  1. Configure Nodes to Authenticate to Private Repo:
    > Nodes should be configured to Authenticate to Private Repo. 
  2. Pre-pull Images and save in Chache. 
    > All pods in the cluster can use any images cached.
  3. ImpagePUll Secrets at POD level. 
    > Pods with the secret key only can pull the image. 

  What can containers see ?
  Filesystem:
  - Images
  - volumes 
  Hostname 
  POD 
   - Pod name, user-defined environment variables from the pods defenition. 
  Services 
   - List of all services running when containers are created, in the form of environment variables.

  Setting up environment variables in Container level:
  ---------------------------------------------------
  # cat containerenv.yaml 
    apiVersion: "v1"
    kind: Pod
    metadata:
      name: envvariable 
      labels:
        app: demo
        env: test 
    spec: 
      containers:
        - name: envvariable 
          image: nginx
          env:
          - name: DEMO_MY_VARIABLE1
            value: "my first variable"  
          - name: DEMO_MY_VARIABLE2
            value: "my second variable"  
          volumeMounts:
            - name: secret-volume
              mountPath: /etc/secret-volume
      volumes:
        - name: secret-volume
          secret:
            secretName: test-secret 
   - This will create a container with the env variable defined in the container spec. Verify using below command. 
  # kubectl exec -ti envvariable -- /bin/bash
   -# env | grep -i demo
    DEMO_MY_VARIABLE2=my second variable
    DEMO_MY_VARIABLE1=my first variable

  Downward API: Passing information from pod to container. 
  -------------------------------------------------------
  - DownwardAPI is the way for POD to pass pod specific informaiton to the container. This is used to pass the POD specific information to the container. 
  # cat DownwardAPI.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: kubernetes-downwardapi-volume-example
    labels:
      zone: us-east-1
      cluster: test-cluster1
      rack: rack-22
    annotations:
      build: two
      builder: john-doe
  spec:
    containers:
      - name: client-container
        image: k8s.gcr.io/busybox
        command: ["sh", "-c"]
        args:
        - while true; do
            if [[ -e /etc/podinfo/labels ]]; then
              echo -en '\n\n'; cat /etc/podinfo/labels; fi;
            if [[ -e /etc/podinfo/annotations ]]; then
              echo -en '\n\n'; cat /etc/podinfo/annotations; fi;
            sleep 5;
          done;
        volumeMounts:
          - name: podinfo
            mountPath: /etc/podinfo
            readOnly: false
    volumes:
      - name: podinfo
        downwardAPI:
          items:
            - path: "labels"
              fieldRef:
                fieldPath: metadata.labels
            - path: "annotations"
              fieldRef:
                fieldPath: metadata.annotations

  - Now check the configuration using below command
  # kubectl logs pod/kubernetes-downwardapi-volume-example 
   - Echo command in the container will be captured as the std-out, and should be availble in the logs. 
  # kubectl exec -ti kubernetes-downwardapi-volume-example -- sh
  # ls -alR /etc/podinfo 
   - Could see that the new files are created and they are symbolic links to the directories. This is how the refreshing the metadata takes place.

  How conainers react to Lifecycle events:
  ---------------------------------------
  Lifecycle Hooks:
  - PostStart:
    - Called immediately after container is created 
    - cannot pass any parameters. 
  - PreStop:
    - Runs before container terminates.
    - Must complete before the conatiner can be stopped.

  Hooks are implemented by using HookHandlers. 
  Two types:
   1. Exec 
     - Can be used to run specific shell command. 
   2. HTTP 
     - can send a http request to a specific endpiont of container.

  Demo of Lifecycle hooks for containers:
  # cat lifecycledemo.yaml 
    apiVersion: "v1"
    kind: Pod
    metadata:
      name: lifecycledemo
      labels:
        app: demo
        env: test
    spec:
      containers:
        - name: lifecycledemo
          image: nginx
          lifecycle:
            postStart:
              exec:
                command: ["/bin/sh", "-c", "echo hello from the postStart Handler > /usr/share/message"]
            preStop:
              exec:
                command: ["/usr/sbin/nginx", "-s" "quit"]

    - Now login and check if the preStart hook is executed. 
  # kubectl exec -ti lifecycledemo -- /bin/bash
  # cat /usr/share/message 
  hello from the postStart Handler

  Init Containers:
  ---------------
   This is kind of startup script, like they run before the app containers are start.
   - Init container run to complete. 
   - Runs in serial. Only starts the container after previous one is finished. 
   - If init container fails, K8 will repeatedly restarts the container until it is successful. 
   - Usually used to run utilities before app container starts. 
   
   How to use init container to setup the state of the pod: 
   -----------------------------------------------------
   In this example, we will create an init container inside a pod which will pre-populate data for the side container running in the same pod. 
   > init runs first to download the index.html to a workdir -> the workdir is emptyDir volume shared between all containers in the pod. The nginx container serves data from this directory 

  # catinitpod.yaml
    apiVersion: "v1"
    kind: Pod
    metadata:
      name: initpod
      labels:
        app: demo
        env: test
    spec:
      containers:
        - name: initpodnginx
          image: nginx
          ports:
          - containerPort: 80
          volumeMounts:
          - name: workdir
            mountPath: /usr/share/nginx/html 
    # Following is the init container runs duyring pod initialization. 
      initContainers:
      - name: install
        image: busybox
        command:
        - wget
        - "-O"
        - "/work-dir/index.html"
        - http://kubernetes.io
        volumeMounts:
        - name: workdir
          mountPath: /work-dir
      dnsPolicy: Default
      volumes:
      - name: workdir
        emptyDir: {}     

  # kubectl apply -f initpod.yaml   


   Conainer Probes with in a POD:
   ------------------------------
     This is to check the status of the individual containers in a POD. 
     Kubletes in nodes dispatches the probes, probes are diagnotics performed periodically on conainters. 

     There are two types of Probes:
    1. Liveness Probes
    ------------------
     If Liveness probes failed, Kubelete assumes that the container is dead and depending on restart policy of the container, it will kick in. 
     Used, if container needs to be killed and restarted on probe failure. 
     Step 1. Set liveness probe 
     Step 2. Specify restart policy of Alway or On-Failure. 
     - Used by kubelet to figure out when a continer needs to be restarted. Used to check the Availability of the Application. 

    Example: # cat livenessProbe.yaml 
      apiVersion: "v1"
      kind: Pod
      metadata:
        name: livenessprobpod
        labels:
          app: demo
          env: test
      spec:
        containers:
        - name: install
          image: k8s.gcr.io/busybox
          args:
          - /bin/sh
          - -c
          - touch /tmp/health; sleep 30; rm -rf /tmp/health; sleep 600
          livenessProbe:
            exec:
              command:
              - cat 
              - /tmp/health
            # initialDelay in sec tells kubelete to start probe only after the 5 sec.
            initialDelaySeconds: 5
            # kubelete will perform the probe in every 5 secs.
            periodSeconds: 5    

    Create and describe the pod to check the status. 
    # kubectl describe pods livenessprobpod

    Example: Use HTTP for liveness check:
    ------------------------------------
      Below pod has an application running for few minutes and coded to get crahsed in 10s. So it is expected to see that HTTP Liveness probe is failing after 10sec and restarting the container. 
    apiVersion: "v1"
    kind: Pod
    metadata:
      name: livenesshttp
      labels:
        app: demo
        env: test
    spec:
      containers:
      - name: liveness
        image: k8s.gcr.io/liveness
        args:
        - /server
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            httpHeaders:
            - name: X-Custom-Header
              value: Awesome
          # initialDelay in sec tells kubelete to start probe only after the 5 sec.
          initialDelaySeconds: 3
          # kubelete will perform the probe in every 5 secs.
          periodSeconds: 3

    Create and describe the pod to check the status. 
    # kubectl describe pods livenesshttp
    

    Example: Use TCP for liveness check:
    -----------------------------------
      apiVersion: "v1"
      kind: Pod
      metadata:
        name: livenesstcp
        labels:
          app: demo
          env: test
      spec:
        containers:
        - name: livenesstcp
          image: nginx
          readinessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 5
            periodSeconds: 10
          livenessProbe:
            tcpSocket:
              port: 80
            initialDelaySeconds: 15
            periodSeconds: 20 

    Create and describe the pod to check the status. 
    # kubectl describe pods livenesstcp

    2. Readiness Probes 
    -------------------
     Whether containers are ready for service request. If the probes are failed, the Endpoint object will be removed from all the Services. 
     Used if the traffic needs to be send only after the probe is succeeded in the container.
     - Used to check when the conainer is ready to accept the traffic. 
    Example for readiness probe: 
    --------------------------
      apiVersion: "v1"
      kind: Pod
      metadata:
        name: readinessprobe
        labels:
          app: demo
          env: test
      spec:
        containers:
        - name: readinessprobe
          image: nginx
          ports:
          - containerPort: 8080
          readinessProbe:
            tcpSocket:
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
  
  Pod Preset:
  ----------
   Helps to inject specific information to pods, while they are creating. 
   - Works similarly like labels and selectors. 

  Pod Priorities:
  --------------
   Helps to indicate the priority of a pod over than other. So this helps in scheduling the high priority. 
   - This is defined by creating a PriorityClass object.  
   - Then in po refer the priorityClassName to the priority object created. 
   - The priority influences
     - Helps to jump the Scheduling Order. 
   - Preemption: 
     Scheduling logic may pre-empt lower priority pod to make way for the high priority pode.

  Statis Pods
  ------------
  On nodes:
   # cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
     KUBELETE_ARGS="--CLUSTER-DNS=10.96.0.10 --CLUSTER-DOMAIN=cluster.local --pod-manifest-path=/etc/kubelet.d/"
   # cat /etc/kubelet.d/static-web.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: static-web
  labels:
    role: myrole
spec:
  containers:
    - name: web
      image: nginx
      ports:
        - name: web
          containerPort: 80
          protocol: TCP

    # restart the service 
    # systemctl restart kubelet
    #  systemctl daemon-reload

  =======================
   Affinity & Anti-Affinity
  =======================
  Tells pods where they should be scheduled. Deals with attracting pods to nodes. 
  Attracts pods to Nodes. 
  It has more control over which Nodes needs to be chosen for a Pod. 
  Example: Choose an instance type of nodes for the pods running. 

 Assign a Pod to specific Node: 
 -----------------------------
  Decision is usually made by the Kube-Scheduler. 
  But incase if you need 
  - specific Hardware
  - colocate pods to same nodes
  - for High-Availability by forcing pods to different nodes.
  Two methods, 
   1. nodeSelector:
     - Tag nodes with labels and use that label on pods to select the node.  
     - Add nodeSelector to pod template
     - Pods will only reside on nodes tha are selected by nodeSelector.

   Note: Every node is pre-provisioned with labels as below, which can be made use to set nodeSelector, Affinity or Anti-Affinity.
   Example:
   Labels:          beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/instance-type=t2.xlarge
                    beta.kubernetes.io/os=linux
                    failure-domain.beta.kubernetes.io/region=us-east-1
                    failure-domain.beta.kubernetes.io/zone=us-east-1a
                    kubernetes.io/hostname=ip-192-168-101-236.ec2.internal

  Associating Pods with Nodes using nodeSelector:
  -----------------------------------------------
  Use the default label or create a new Label so that the pods can be scheduled. 
  # kubectl label nodes ip-192-168-143-231.ec2.internal nodeos=ubuntu
  # kubectl get nodes --show-labels
    - Check the labels 
  # cat nodeslector.yaml 
    apiVersion: "v1"
    kind: Pod
    metadata:
      name: nodeselect 
      labels:
        app: demo
        env: test 
    spec: 
      nodeSelector:
        nodeos: ubuntu
      containers:
        - name: envvariable 
          image: nginx
          env:
          - name: DEMO_MY_VARIABLE1
            value: "my first variable"  
          - name: DEMO_MY_VARIABLE2
            value: "my first variable"
  # kubectl apply -f nodeslector.yaml  
   - This will make sure that the pod is sheduled on a node that has the label Key = nodeos and value = ubuntu 

    2. Affinities and Anti-Affinities 
   This can be catagorized to two 
    - 1. Node affinity 
      Steer pod to node. 
    - 2. Pod Affinity
      Steer pod towards a pod, to run two pods together. 
      It can also be configured to set the pods away from each other.  

Example:
  Label your node as being located in availability zone 1:
  # kubectl label node chadcrowell1c.mylabserver.com availability-zone=zone1

  Label your node as dedicated infrastructure:
  # kubectl label node chadcrowell1c.mylabserver.com share-type=dedicated

  Here is the YAML for the deployment to include the node affinity rules:
    apiVersion: extensions/v1beta1
    kind: Deployment
    metadata:
      name: pref
    spec:
      replicas: 5
      template:
        metadata:
          labels:
            app: pref
        spec:
          affinity:
            nodeAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
              - weight: 80
                preference:
                  matchExpressions:
                  - key: availability-zone
                    operator: In
                    values:
                    - zone1
              - weight: 20
                preference:
                  matchExpressions:
                  - key: share-type
                    operator: In
                    values:
                    - dedicated
          containers:
          - args:
            - sleep
            - "99999"
            image: busybox
            name: main

 PodAffinity
 -----------

 # kubectl label po mypod security=S1
 # cat PodAffinity.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: podaff
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: kubernetes.io/hostname
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
   
  # kubectl apply -f PodAffinity.yaml 
   Creates the pod podaff at the same node where the pod mypod is running. 


  =======================
  Taints and Tolerations:
  =======================
  Taints:
      Prevent pod from running on a worker node. 
      Reserve a pod to a node. 
      Repells pods from Nodes.
      Can be used if pods needs to run on a dedicated node. Like taint the node for all pods, and only add toleration for the pods which needed to make use of the Node. 

      Types of Taints:
       NoSchedule: 
        With NoSchedule taint, future pods will not be scheduled on this node but any exisiting pods will continue to run as normal. 
        Example: 
        Taints:             node-role.kubernetes.io/master:NoSchedule
       PreferNoSchedule:
        With PreferNoSchedule, is a soft version of NoSchedule, where future pods will TRY TO AVOID this node, but can still run here if no other nodes are available. 
       NoExecute:
         With NoExecute, even the pods already running on this Node will be stopped and evicted from the node. The only Exception to this is if the pod has a tolderation to the NoExecute Taint. 
    
    Example: 
      Apply Taint to Steer pods away from a node. 
      #  kubectl taint nodes ip-192-168-252-145.ec2.internal key=value:NoSchedule

      Now scale up the deployment and see how the pods are being  allocated. 
      # kubectl scale deployments/nginx --replicas=10
      # kubectl get pods -o wide

     To remove the taint added:
      # kubectl taint nodes ip-192-168-252-145.ec2.internal key:NoSchedule-

  Tolerations:
   Pods can continue to run on a Node even the taint is set. 
  Example:
   Create deployment to Tolerate the Taint:
   # kubectl create deployment tolerationdep --image=nginx --dry-run -o yaml  
    Copy the Yaml and edit with the toleration as below. 
   # cat toleratedep.yaml 
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        creationTimestamp: null
        labels:
          app: tolerationdep
        name: tolerationdep
      spec:
        replicas: 10
        selector:
          matchLabels:
            app: tolerationdep
        strategy: {}
        template:
          metadata:
            creationTimestamp: null
            labels:
              app: tolerationdep
          spec:
            containers:
            - image: nginx
              name: nginx
              resources: {}
            tolerations:
            - key: "key"
              operator: "Equal"
              value: "value"
              effect: "NoSchedule"
      status: {}
   
   # kubectl apply -f toleratedep.yaml 
    This will provision the pods on a Tainted node. Important part is that the Key and Value shoudl match. 

    Example:
    # kubectl describe pod/kube-proxy-825nw -n kube-system 
    Tolerations:     
                 CriticalAddonsOnly
                 node.kubernetes.io/disk-pressure:NoSchedule
                 node.kubernetes.io/memory-pressure:NoSchedule
                 node.kubernetes.io/network-unavailable:NoSchedule
                 node.kubernetes.io/not-ready:NoExecute
                 node.kubernetes.io/unreachable:NoExecute
                 node.kubernetes.io/unschedulable:NoSchedule
   Kubeprox needs to be running on all nodes including master and it has all tolerations.

  




  =======================
   Pod Networks and Flannel
  =======================
    - All containers can communicate each other without a NAT. 
    - All nodes can communiate without NAT. 
    - The IP that a container sees itself as is the same IP that others sees as. 
    - Containers with in the same POD communicate with the ports. 
    Flannel is a simple and easy way to condigue a layer 3 network fabric for Kubernetes



    =======================
    Install and Configure Kubernetes Cluster on Ubuntu:
    =======================
    - Get ubuntu 16.04 
    Master: 
    1. Installing packages:
     # sudo apt-get update  && apt-get install apt-transport-https # -> Helps to comminicate between control plane to pods. 
     # apt-get install docker.io && systemctl start docker
     # systemctl enable docker
     # curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
     # cat /etc/apt/sources.list.d/kubernetes.list 
      deb http://apt.kubernetes.io/ kubernetes-xenial main
     # apt-get update && apt-get install -y kubelet kubeadm kubectl kubernetes-cni

    2. Bootstrap Master:
     # kubeadm init 
       This will run validation check. 
       Generates the self signed certificate to setup identity for diff components of master server. 
       A certificate of the API server is created and signed. 
       Setup the kubeconfig file under /etc/kubernets/ for kubelete, controller manager and the scheduler. 
       API server, controller manager and scheduler are running inside the PODs.  Static pod manifest for these pods in control plane is setup by the master.
       It applies label to the master node. 
       kubeadm will generate a token that can be specified by nodes to join the master. 
       Setus up CoreDNS and kube-proxy. 
       Finally helpful messages to setup command to setup master and then commands to join nodes to master. 
   -- output --
        Your Kubernetes control-plane has initialized successfully!

      To start using your cluster, you need to run the following as a regular user:

        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config

      You should now deploy a pod network to the cluster.
      Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
        https://kubernetes.io/docs/concepts/cluster-administration/addons/

      Then you can join any number of worker nodes by running the following on each as root:

      kubeadm join 192.168.230.172:6443 --token x22wz7.2fyzpinmfuv1f08d \
          --discovery-token-ca-cert-hash sha256:e80e50f9aabc2a01296fb48cb422753f0fe0daac2ae18b70bc01241ceb56592d 
   -- out put end --
     # mkdir -p $HOME/.kube
     # sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     # sudo chown $(id -u):$(id -g) $HOME/.kube/config
     # kubectl cluster-info
     Now setup the flannel node network for Pod Network. 
     # kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
      podsecuritypolicy.extensions/psp.flannel.unprivileged created
      clusterrole.rbac.authorization.k8s.io/flannel created
      clusterrolebinding.rbac.authorization.k8s.io/flannel created
      serviceaccount/flannel created
      configmap/kube-flannel-cfg created
      daemonset.extensions/kube-flannel-ds-amd64 created
      daemonset.extensions/kube-flannel-ds-arm64 created
      daemonset.extensions/kube-flannel-ds-arm created
      daemonset.extensions/kube-flannel-ds-ppc64le created
      daemonset.extensions/kube-flannel-ds-s390x created

    Make sure that below is set else network will fail:
    /etc/kubernetes/manifests/kube-controller-manager.yaml:    - --allocate-node-cidrs=true
    /etc/kubernetes/manifests/kube-controller-manager.yaml:    - --cluster-cidr=192.168.0.0/16
    /etc/kubernetes/manifests/kube-controller-manager.yaml:    - --node-cidr-mask-size=24
     This will make sure to get cluster cidr as below:
    #  kubectl cluster-info dump | grep -m 1 cluster-cidr

   Worker Node:
    1  apt-get update
    2  sudo apt-get install apt-transport-https
    3  apt-get install docker.io 
    4  systemctl start docker
    5  systemctl enable docker
    6  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add
    7  cat /etc/apt/sources.list.d/kubernetes.list 
        deb http://apt.kubernetes.io/ kubernetes-xenial main
    8  apt-get update && apt-get install -y kubelet kubeadm kubectl kubernetes-cni
    9. kubeadm join 192.168.230.172:6443 --token x22wz7.2fyzpinmfuv1f08d \
          --discovery-token-ca-cert-hash sha256:e80e50f9aabc2a01296fb48cb422753f0fe0daac2ae18b70bc01241ceb56592d 

    - Now on master run 
    # kubect get nodes. 


   
  

  =======================
   IAM Roles for pods
  =======================
   kiam, kube2iam, Valut 

  =======================
   Health check and Probes 
  =======================
    Health checks are called probes in pods.
   There are two kinds of probs. 
     Liveness -> Checks if the container is function. Container is restarted if the Liveness is failed. 
     Readiness Probes: -> Check if the container applicaiton is ready to recieve traffic. Timeout and 5xx will be avoided. K8 service will not send traffic unless readiness probe is up. 

  =======================
   Measuring CPU and Memory 
  =======================
   CPU:
    Limits and requests for CPU resources are measured in cpu units. One cpu, in Kubernetes, is equivalent to:
    1 AWS vCPU
    1 GCP Core
    1 Azure vCore
    1 IBM vCPU
    1 Hyperthread on a bare-metal Intel processor with Hyperthreading

Fractional requests are allowed. A Container with spec.containers[].resources.requests.cpu of 0.5 is guaranteed half as much CPU as one that asks for 1 CPU. The expression 0.1 is equivalent to the expression 100m, which can be read as “one hundred millicpu”.

 Memory:
 Limits and requests for memory are measured in bytes. You can express memory as a plain integer or as a fixed-point integer using one of these suffixes: E, P, T, G, M, K. You can also use the power-of-two equivalents: Ei, Pi, Ti, Gi, Mi, Ki. For example, the following represent roughly the same value:
  128974848, 129e6, 129M, 123Mi

 Overprovisioning will kill the container if the resource utilization is high. 



Allocating Pods with CPU and Memory:
-----------------------------------

  Check current CPU and Memory on nodes. 
  # kubectl describe node ip-192-168-223-238
    Capacity: # ---> This is the total capacity of the Node 
  cpu:                2
  ephemeral-storage:  8065444Ki
  hugepages-2Mi:      0
  memory:             8173692Ki
  pods:               110
  Allocatable: # ---> Allocatable capacity for pods. 
  cpu:                2
  ephemeral-storage:  7433113179
  hugepages-2Mi:      0
  memory:             8071292Ki
  pods:               110

  Create the pod YAML for a pod with requests:
  # cat resource-pod1.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: resource-pod1
  spec:
    nodeSelector:
      kubernetes.io/hostname: "ip-192-168-223-238"
    containers:
    - image: busybox
      command: ["dd", "if=/dev/zero", "of=/dev/null"]
      name: pod1
      resources:
        requests:
          cpu: 800m
          memory: 20Mi

  Create the requests pod:
  # kubectl create -f resource-pod1.yaml

  View the pods and nodes they landed on:
  # kubectl get pods -o wide

  The YAML for a pod that has a large request:
  # cat resource-pod2.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: resource-pod2
  spec:
    nodeSelector:
      kubernetes.io/hostname: "ip-192-168-223-238"
    containers:
    - image: busybox
      command: ["dd", "if=/dev/zero", "of=/dev/null"]
      name: pod2
      resources:
        requests:
          cpu: 1000m
          memory: 20Mi

  Create the pod with 1000 millicore request:
  # kubectl create -f resource-pod2.yaml

  See why the pod with a large request didn’t get scheduled:
  # kubectl describe resource-pod2

  Look at the total requests per node:
  # kubectl describe nodes ip-192-168-223-238

  Delete the first pod to make room for the pod with a large request:
  # kubectl delete pods resource-pod1

  Watch as the first pod is terminated and the second pod is started:
  # kubectl get pods -o wide -w

  The YAML for a pod that has limits:
  # cat limited-pod.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: limited-pod
  spec:
    containers:
    - image: busybox
      command: ["dd", "if=/dev/zero", "of=/dev/null"]
      name: main
      resources:
        limits:
          cpu: 1
          memory: 20Mi

  Create a pod with limits:
  # kubectl create -f limited-pod.yaml

  Use the exec utility to use the top command:
  # kubectl exec -it limited-pod top



  =======================
   Downward API - Introspection  
  =======================
  Sometimes application needs to expose pod metadata for pod application. Useful when pods application needs to make decissions based on where the pods are scheduled. 
   You can expose the same via environment variables. 
   Refer:
   https://kubernetes.io/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/#capabilities-of-the-downward-api


  =======================
  Volumes: 
  =======================
   Containers running inside the pods are ephemeral. It can lose anything stored locally during crash or respawn of the container. Storages lasts as long as the life time of a POD.  
    
   - Volumes are used to save the data processed by pods. 
   - Share state/files with multiple containers in a pod.
   - Volumes are going to leave until the PODs are available, to make sure its is available when the containers are restarted. 
   - If POD is deleted, then the Volumes will be deleted as well. Use Persistent Volumes if you want to have the data saved even the Pod is expired.  
   
   > There are Persistent and non-Persistent volumes:

   -------------------
   Common Volume Types:
   -------------------
   1 - emptyDir
   ------------
     - Created when a pod is assigned to a node and available as long as the pod is running on the node. When pod is removed the condent of the volume is lost. There is no isolation between container or pods and no limit on size that the volume can consume. 
     - It shares space between containers.
     - Non persistent. Share storage between containers in the pod.  
     - Freshly created when the pod is launched. 
     - Can be used as a check point, for when the container is crashed, 
     - it has the same life time as the POD. 
     - emptyDir Example:
      apiVersion: v1
      kind: Pod
      metadata:
        name: redis
      spec:
        containers:
        - name: redis
          image: redis
          volumeMounts:
          - name: redis-storage
            mountPath: /data/redis
        volumes:
        - name: redis-storage
          emptyDir: {}
      # test by 
      #  kubectl exec -ti redis -- /bin/bash 
        Then touch a file inside /data/redis and kill the init 1 process, It should respawn new container, and loging in back should see the data persist. 

   2 - ConfigMap
   ------------
       - It works closely with the configMap resources. Together the configMap resource and volume injects the parameters or configuration to the Containers. 

      Create ConfigMap
      ----------------
       # echo Hello > configmapfile.txt 
       # echo Hi > configmapfile2.txt 
       # kubectl create configmap fmap --from-file=configmapfile.txt --from-file=configmapfile2.txt 
         Now the data in the files are added to the cluster meta store embeded inside the configmap object. 
       # kubectl get configmaps fmap -o yaml
      apiVersion: v1
      data:
        configmapfile.txt: |
          hello
        configmapfile2.txt: |
          Hi
      kind: ConfigMap
      metadata:
        creationTimestamp: "2019-06-07T04:05:56Z"
        name: fmap
        namespace: default
        resourceVersion: "27607122"
        selfLink: /api/v1/namespaces/default/configmaps/fmap
        uid: 8ca86469-88d9-11e9-b153-1222897cc046
      
        - This will show the details of config map. Below snippet can be used at the pod level.  
        env:
        - name: YOUR_NAME
          valueFrom:
            configMapKeyRef:
              name: kiran
              key: name
        - name: GREETINGS_FROM_FILE
          valueFrom:
            configMapKeyRef:
              name: fmap
              key: configmapfile.txt

       Use configMap in a pod 
       ----------------------
       1. Create a configmap:
       # kubectl create configmap myname --from-literal=name=kiran
       # kubectl get configmap/kiran -o yaml
        apiVersion: v1
        data:
          name: myname
        kind: ConfigMap
        metadata:
          creationTimestamp: "2019-06-07T04:17:50Z"
          name: myname
          namespace: default
          resourceVersion: "27610550"
          selfLink: /api/v1/namespaces/default/configmaps/myname
          uid: 36bb4c29-88db-11e9-821d-029f62b75b98
       # Create pod. 
        apiVersion: v1
        kind: Pod
        metadata:
          name: configmaptest
        spec:
          containers:
            - name: configmap
              image: nginx 
              env:
                - name: YOUR_NAME
                  valueFrom:
                    configMapKeyRef:
                      name: myname
                      key: name
       # Login to the pod and should be able to see the value of env variable updated. 

       Example 2:
      Create a ConfigMap with two keys:
      # kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2

      Get the YAML back out from the ConfigMap:
      # kubectl get configmap appconfig -o yaml

      The YAML for the ConfigMap pod:
      # cat configmappod.yaml 
      apiVersion: v1
      kind: Pod
      metadata:
        name: configmap-pod
      spec:
        containers:
        - name: app-container
          image: busybox:1.28
          command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
          env:
          - name: MY_VAR
            valueFrom:
              configMapKeyRef:
                name: appconfig
                key: key1

      Create the pod that is passing the ConfigMap data:
      # kubectl apply -f configmap-pod.yaml

      Get the logs from the pod displaying the value:
      # kubectl logs configmap-pod


       Making use of configMap and mounting as volume in pod. 
       ------------------------------------------------------
       > Refer to the configmaps created in above steps. 
       # Pod Configuration yaml example:
      apiVersion: v1
      kind: Pod
      metadata:
        name: configmaptest
      spec:
        containers:
          - name: configmap
            image: nginx
            env:
              - name: YOUR_NAME
                valueFrom:
                  configMapKeyRef:
                    name: myname
                    key: name
              - name: GREETINGS_FROM_FILE
                valueFrom:
                  configMapKeyRef:
                    name: fmap
                    key: configmapfile.txt
            volumeMounts:
            - name: configmapfilevolume
              mountPath: /etc/config1
            - name: configmapkeyvalue
              mountPath: /etc/config2
        volumes:
          - name: configmapfilevolume
            configMap:
              name: fmap
          - name: configmapkeyvalue
            configMap:
              name: myname

        - This will make sure the configMap files are mounted and available at the /etc/config? locations.

      Example 2:
      # kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2

      The YAML for a pod that has a ConfigMap volume attached:
      # cat configMapVolume.yaml 
      apiVersion: v1
      kind: Pod
      metadata:
        name: configmap-volume-pod
      spec:
        containers:
        - name: app-container
          image: busybox
          command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
          volumeMounts:
            - name: configmapvolume
              mountPath: /etc/config
        volumes:
          - name: configmapvolume
            configMap:
              name: appconfig

      Create the ConfigMap volume pod:
      # kubectl apply -f configmap-volume-pod.yaml

      Get the keys from the volume on the container:
      # kubectl exec configmap-volume-pod -- ls /etc/config

      Get the values from the volume on the pod:
      # kubectl exec configmap-volume-pod -- cat /etc/config/key1



   3 - gitRepo
   -----------
       - it mounts the emptyDir and clones the gitRepo into the pod, so that the containers can use. 

   4 - secret
   ---------- 
       - Helps to pass sensitive informaton to container. # kubectl create secret. Then mount the files to the POD. It is encoded value of string in base64. Mounts in memory space. 
      Use Secret to pass sensitive information to POD. 

      Create Secret using base64. 
      --------------------------
      # Generate base64 string 
      # echo -n "my-app" | base64
      # echo -n "password" | base64

      # Create secret Object: 
      apiVersion: v1
      kind: Secret
      metadata:
        name: test-secret
      data:
        username: bXktYXBw
        password: cGFzc3dvcmQ=
      
      # kubectl create -f secret.yaml 
      # kubectl get secret/test-secret
      NAME          TYPE     DATA   AGE
      test-secret   Opaque   2      15s
      
      # create a POD to access the secret. 
      apiVersion: "v1"
      kind: Pod
      metadata:
        name: mypod-secret
        labels:
          app: demo
          env: test 
      spec: 
        containers:
          - name: nginx-secret
            image: nginx
            volumeMounts:
              # name must match the volume name below. 
              - name: secret-volume
                mountPath: /etc/secret-volume
        volumes:
          - name: secret-volume
            secret:
              secretName: test-secret 

      Create Secret from a file:
      -------------------------
      Generate the file:
      # echo -n "blingothar" > username.txt
      # echo -n "blingothar123" > password.txt
      # kubectl create secret generic sensitive --from-file=./username.txt --from-file=./password.txt 
      # kubectl get secrets
      # kubectl describe secrets/sensitive
      Now create the pod:
      apiVersion: "v1"
      kind: Pod
      metadata:
        name: secretfilepod
        labels:
          app: demo
          env: test 
      spec: 
        containers:
          - name: nginxsecretpodfile
            image: redis
            env:
              - name: SECRET_USERNAME
                valueFrom:
                  secretKeyRef:
                    name: sensitive
                    key: username.txt 
              - name: SECRET_PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: sensitive
                    key: password.txt 
        restartPolicy: Never       
      # kubectl apply -f secretfilepod.yaml
      # kubectl describe pods/secretfilepod
         - This will show the environment variables set. Also can check the environment variables by login to the container. 

      Example 2:
      The YAML for a secret:
      # cat appsecret.yaml 
      apiVersion: v1
      kind: Secret
      metadata:
        name: appsecret
      stringData:
        cert: value
        key: value

      Create the secret:
      # kubectl apply -f appsecret.yaml

      The YAML for a pod that will use the secret:
      # cat appcontainer.yaml 
      apiVersion: v1
      kind: Pod
      metadata:
        name: secret-pod
      spec:
        containers:
        - name: app-container
          image: busybox
          command: ['sh', '-c', "echo Hello, Kubernetes! && sleep 3600"]
          env:
          - name: MY_CERT
            valueFrom:
              secretKeyRef:
                name: appsecret
                key: cert

      Create the pod that has attached secret data:
      # kubectl apply -f secret-pod.yaml

      Open a shell and echo the environment variable:
      # kubectl exec -it secret-pod -- sh
      # echo $MY_CERT

      The YAML for a pod that will access the secret from a volume:
      # cat secretVolume.yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: secret-volume-pod
      spec:
        containers:
        - name: app-container
          image: busybox
          command: ['sh', '-c', "echo $(MY_VAR) && sleep 3600"]
          volumeMounts:
            - name: secretvolume
              mountPath: /etc/certs
        volumes:
          - name: secretvolume
            secret:
              secretName: appsecret

      Create the pod with volume attached with secrets:
      # kubectl apply -f secret-volume-pod.yaml

      Get the keys from the volume mounted to the container with the secrets:
      # kubectl exec secret-volume-pod -- ls /etc/certs


   5 - hostPath
   ------------
        Mounts a file/dir from host nodes filesystems into the pods, such as docker internals. When using using hostpath the pods of same config can behave differently on different nodes. Files and directory created under the host is only root writable and needs to require a privilaged container. Can be confiured to access Block devices or sockerts of underlying hosts.
    
    - Other Volume types 
    1. 5. Varoius other cloud Provides, awsElasticBlockStore, azure disk, etc. 
    awsElasticBlockStore volume:
      The condents of EBS volumes is preserved and unmounted when the pod is removed. Node should be ec2 instance and should be in same region and can only be mounted to a single instance. 

     - Exammple:
      apiVersion: v1
      kind: Pod
      metadata:
        name: ebsredis
      spec:
        containers:
        - name: redis
          image: redis
          volumeMounts:
          - name: redis-storage
            mountPath: /data/redis
        volumes:
        - name: redis-storage 
          awsElasticBlockStore:
            volumeID: vol-0933587f596f1964c
            fsType: ext4
          
          - This will make sure that the volume id 'vol-0933587f596f1964c' is attached to the Underlying node, and made available to the PODs. Now the data is persistent even after the pod is perished. Once the Pods are deleted the volume is dettached from the node and makes available for next pod. 

    2. Local 
       These are mounted local storage devices (such as disk, or local directory). To use this volumes Persistent node affinity is required so that the scheduler can put pods on correct nodes.

  > Pod Configuration should have the spec to Create Volume, to specify which volume to be used -> Then mount that into the Container. 

   Persistent Volume 
   -----------------
    The normal volume type configuration (awsElasticBlockStore) does needs the knowledge of platform on which the volume is created, and need to feed the volumeID information while creating a POD/Deployment. The Persistent Volume does allows remove this dependency and can keep the configuration usable accross multuple cloud Vendor.

   Persistent Volumes PV:
    - are piece of storage provisioned by and administrator. This is a resource with in the cluster. A volume plugin with a lidecycle independent of any individual pod that uses the PV. 

   Persistent Volume Claim PVC:
    - Is a request for storage by a user. The request can be a specific size or access mode (read only, read write etc). 
    - Request of access to make use of the PV. The claims bonds the volume to the claim. Doesn't matter what type of the volume is inside. 
    - Once a claim is bound, this can be used on a POD. 

  
  Lifecycle of PV and PVC:
   Static or Dynamic. 
    Static: 
    Dynamic:    
    - Lives longer than the pods, the data gets preserved even the pods are deleted. 
    - They are low level object like Node. 
    - Persistent Volumes needed to be pre-provisioned by the administrator. 
    TWO Types of Persistent volume provisioning:
    Static:
     Administrator pre-creates the volumes. 
    Dynamic: 
     Containers need to file a Persistent Volume Claim. This claim triggers dymanic provision of a volume. This provisioning is based on a property called StorageClass. Persistent volume claim must request a StorageClass and Adminisitrator must have created and configured the storageclass in advance of claiming the Storage class to work. 

  Access Modes:
  -------------
   - Specifies an PV access wheather its a read or write access by single node or multiple node at same time.
     
   - spec.accessModes.ReadWriteOnce - Only a single node can mount and write to the volumes. 
   - spec.accessModes.ReadOnlyMany - Alloes multiplenodes to mount and read the data from the volume. 
    - This is the mount capability of a Node not the pods. 

  StorageObjects:
  --------------
  Storage Object in use Protection enables. 
  - If pod is deleted the PVC are not removed pre-metuarely. 
  #  kubectl describe pv task-pv-volume
    Finalizers:      [kubernetes.io/pv-protection]

  #  kubectl describe pvc 
    Finalizers:    [kubernetes.io/pvc-protection]
   - If we delete a pvc, it will be in the status of termination. However it will still bound to the pod and will only get removed if the pod is getting deleted. 


    Static: 
    -------
    Below configuration creates the volume for pods from the Worker Nodes path /mnt/data. 

    The YAML for a PersistentVolume object in Kubernetes:
    # cat persistentvol.yaml 
    kind: PersistentVolume
    apiVersion: v1
    metadata:
      name: task-pv-volume
      labels:
        type: local
    spec:
      storageClassName: manual
      capacity:
        storage: 1Gi
      accessModes:
        - ReadWriteOnce
      hostPath:
        path: "/mnt/data"

    # cat pvclaim.yaml 
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: task-pv-claim
    spec:
      storageClassName: manual
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi

    # cat pvnginx.yaml 
    kind: Pod
    apiVersion: v1
    metadata:
      name: task-pv-pod
    spec:
      volumes:
        - name: task-pv-storage
          persistentVolumeClaim:
            claimName: task-pv-claim
      containers:
        - name: task-pv-container
          image: nginx
          ports:
            - containerPort: 80
              name: "http-server"
          volumeMounts:
            - mountPath: "/usr/share/nginx/html"
              name: task-pv-storage


    Dynamic Storge/Volume provisioning on AWS EBS:
    ------------------------------------------
      1. Create a storage class, that can provision the AWS EBS Volume: 

    # cat StorageClass.yaml
      kind: StorageClass
      apiVersion: storage.k8s.io/v1
      metadata:
        name: gp2
        annotations:
          storageclass.kubernetes.io/is-default-class: "true"
      provisioner: kubernetes.io/aws-ebs
      parameters:
        type: gp2
        fsType: ext4  
      
      2. Create a PVC that provisions the Persistent Volume automatically:
    # cat StorageClassPVC.yaml
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: ebsvolume-claim
      spec:
        storageClassName: gp2
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 3Gi

      3. Create a POD that make use of the PVC and mounts it. End result will have a pod with 3 Gig attached to /mnt/test 
    # cat StorageClassPod.yaml
      kind: Pod
      apiVersion: v1
      metadata:
        name: ebs-pod
      spec:
        volumes:
          - name: pod-ebs-storage
            persistentVolumeClaim:
            claimName: ebsvolume-claim 
        containers:
          - name: kms-pod-container
            image: nginx
            ports:
              - containerPort: 80
                name: "http-server"
            volumeMounts:
              - mountPath: "/mnt/test"
                name: pod-ebs-storage

    Dynamic Storge/Volume provisioning AWS EBS using KMS encryption:
    ---------------------------------------------------------------
     Same like Dynamic Prov, only difference is that set the attribute of kmsKeyId in parameters. As the reclaimPolicy is set to retain, it will retain the volume and made use for next claim. 
    # cat encryptedStorage.yaml
      apiVersion: storage.k8s.io/v1
      kind: StorageClass
      metadata:
        name: kmsencrypt 
      parameters:
        encrypted: "true"
        fsType: ext4
        kmsKeyId: arn:aws:kms:us-east-1:XXXXXXXXX:key/XXXX-XXXX-XXXX-XXXX-XXXXXXX 
        type: gp2
      provisioner: kubernetes.io/aws-ebs
      reclaimPolicy: Retain
      volumeBindingMode: Immediate

    # cat encryptedStoragePVC.yaml
      kind: PersistentVolumeClaim
      apiVersion: v1
      metadata:
        name: kmsvolume-claim
      spec:
        storageClassName: kmsencrypt
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 3Gi
    
    # cat encryptedStoragePod.yaml
      kind: Pod
      apiVersion: v1
      metadata:
        name: kms-pod
      spec:
        volumes:
          - name: pod-kms-storage
            persistentVolumeClaim:
            claimName: kmsvolume-claim
        containers:
          - name: kms-pod-container
            image: nginx
            ports:
              - containerPort: 80
                name: "http-server"
            volumeMounts:
              - mountPath: "/mnt/test"
                name: pod-kms-storage

  =======================
   AutoScaling
  =======================
   AutoScaling of nodes:
   ---------------------
    Horizontal:
     - By increasing the number of underlying worker nodes for placement of pods. This is implemented through a Cluster AutoScaler.
    Vefrtical:
     - Scaleup by increasing the number of CPU/Mem on same node. There is no current implementation of Vertical AutoScaling of Nodes.

   AutoScaling of Pods:
   ---------------------
    Horizontal:
     - Horizontal AutoScaling of pods by increasing/decreasing the number of pods on the basis of core metrics(Memory/CPU) or custom metrics. This is implemented by a Horizontal Pod AutoScaler. 
    Vertical:
     - Veritical AutoScaling of pods by increasing/decreasing the resource requests (Mem/CPU) for the pod on basis of usage. This is implemented by a Vertical pod AutoScaler and addon-resizer. 

   Cluster AutoScaler:
   ---------------------
    

  =======================
  ConfigMaps: 
  =======================
   Stores condigurations for K8 applicaitons 
   Data is stored in Key Value Pair 
   For sensitive information use Secrets instead of ConfigMaps.

   To create the ConfigMap use below example: 
   ------------------------------------------

  # kubectl create configmap myconfig --from-file=./testmap
  # kubectl get configmaps myconfig -o yaml
  apiVersion: v1
  data:
    testmap: |
      aaa=bbb
      ccc=ddd
  kind: ConfigMap
  metadata:
     *Here, the filename becomes the Key name. Example: testmap. 

  # kubectl create configmap myconfig2 --from-env-file=./testmap2
  # kubectl get configmaps myconfig2 -o yaml
  apiVersion: v1
  data:
    aaa: bbb
    ccc: ddd
  kind: ConfigMap
  metadata:
   *Here the Keys are the values from the file as we used the --from-env-file option.

   Using ConfigMaps for pods:
   --------------------------
  1. Create a configMap:
   #  kubectl create configmap special-config  --from-literal=special.how=very
   # kubectl get configmap special-config -o yaml
      apiVersion: v1
      data:
        special.how: very
      kind: ConfigMap
      metadata:
        creationTimestamp: "2019-03-14T23:29:01Z"
        name: special-config
        namespace: default
  2. Create a Pod to make use of ConfigMap: 
   # cat using-config-map-envvar.yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: configmaptest
      spec:
        containers:
          - name: test-container
            image: k8s.gcr.io/busybox
            command: [ "/bin/sh", "-c", "env" ]
            env:
              - name: SPECIAL_LEVEL_KEY
                valueFrom:
                  configMapKeyRef:
                    name: special-config
                    key: special.how
  3. Verify the configMap
   # kubectl log configmaptest| grep -i level
    SPECIAL_LEVEL_KEY=very


  =======================
  Secrets: 
  =======================
   - A secret is an object intended to hold a small amount of sensitive data. 
   - Data is stored in Key-Value pair 
   - Data is encoded in Base64 and stored. Data in secret is NOT encrypted. 
   Types of Secrets:
    - Docker registry 
    - TLS 
    - Generic
   # kubectl get secrets 
   # kubectl get secrets default-token-vtfvc -o yaml

  1. Create a generic type of Secret 
   # echo -n 'admin' > username.txt
   # echo -n 'mydbpassordisempty' > password.txt
  2. Create the generic secret type from the file.  
   # kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
   # kubectl get secret db-user-pass -o yaml
      apiVersion: v1
      data:
        password.txt: bXlkYnBhc3NvcmRpc2VtcHR5
        username.txt: YWRtaW4=
      kind: Secret
      metadata:
        creationTimestamp: "2019-03-14T23:47:17Z"
        name: db-user-pass
        namespace: default
      type: Opaque
  # echo bXlkYnBhc3NvcmRpc2VtcHR5| base64 --decode
    mydbpassordisempty
  # echo YWRtaW4=| base64 --decode
    admin
  3. Apply secret to a Pod: 
   # cat using-secrets-envvar.yaml 
      apiVersion: v1
      kind: Pod
      metadata:
        name: secrettest
      spec:
        containers:
          - name: test1-container
            image: k8s.gcr.io/busybox
            command: [ "/bin/sh", "-c", "env" ]
            env:
              - name: SPECIAL_LEVEL_KEY
                valueFrom:
                  configMapKeyRef:
                    name: special-config
                    key: special.how
              - name: USERNAME
                valueFrom:
                  secretKeyRef:
                    name: db-user-pass 
                    key: username.txt
              - name: PASSWORD
                valueFrom:
                  secretKeyRef:
                    name: db-user-pass
                    key: password.txt
   # kubectl logs secrettest 

  =======================
  Kube Proxy  
  =======================
    Kube proxy is a process running on each node that takes care of handling traffic to services. The default mode uses IPTables to create local rules to route traffic to the pods that are part of the service. 
  
  =======================
   Controllers                                  
  =======================
     Pod itself does not offer autohealing or scaling. Any type of pod crashes must be handled at the higher level. Controllers runs the desired state check in a loop and matches the value with the actual number. And takes the action necessary to match the actual and desired state.

  The controllers are.
  1. ReplicaSet
  2. Deployment
  3. Replication controller
  4. DaemonSet
  5. StatefulSet
  6. Run-to-Completion Job.

     
   Example: 
    Deployment -> ReplicaSet -> Pod.   

  =======================
  ReplicaSet:
  ======================= 
   - Which encapsulates a pod defenition and the number of replicas of a pod that must be running. It offers scaling and auto healing.  
   - ReplicaSet is an object that ensures that specific number of Pods are running at a given point of time. 
   - ReplicaSet encapsuales a Pod template and also specifies the number of Pods to be running. 
   - We manage ReplicaSet at Deployment Level. 
   - POD information is encapsulated with in the template field.  
   - replicas decides how many pods to be running. Also starts new ones to match the replicas if the pod crahses. 
   - Selector, helps to match pods with the replicaset which is foverned.
   - ReplicaSet couples pods using the Label Selector available under spec -> selector -> matchLabels. 
   - Can Delete a replicaSet without affecting pods. --cascade=false can be used.
   - Can isolate pods from a replicaSet. Changing label of pods can achieve this. 
   - Can change replicaSet which is governing pods on the fly. Creating new replcaSet with same field selector will take over the pods. 

   Example:
    # cat replicaset.yaml
      apiVersion: apps/v1
      kind: ReplicaSet
      metadata:
        name: frontend
        labels:
          app: guestbook
          tier: frontend
      spec:
        # modify replicas according to your case
        replicas: 3
        selector:
          matchLabels:
            tier: frontend
        template:
          metadata:
            labels:
              tier: frontend
          spec:
            containers:
            - name: php-redis
              image: gcr.io/google_samples/gb-frontend:v3

   -> This will create a replicaset with 3 pods running. 
    #  kubectl describe rs/frontend
        Name:         frontend
        Namespace:    default
        Selector:     tier=frontend # --> >> >>> RS searches for the key tier and value as frontend.  
        Labels:       app=guestbook
                      tier=frontend
        Annotations:  <none>
        Replicas:     3 current / 3 desired   # ---> >> >>> Starts three replications. 
        Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
        Pod Template:
          Labels:  app=guestbook
                  tier=frontend
          Containers:
          php-redis:
            Image:        gcr.io/google_samples/gb-frontend:v3
            Port:         <none>
            Host Port:    <none>
            Environment:  <none>
            Mounts:       <none>
          Volumes:        <none>
        Events:
          Type    Reason            Age   From                   Message
          ----    ------            ----  ----                   -------
          Normal  SuccessfulCreate  39s   replicaset-controller  Created pod: frontend-s7rxk
          Normal  SuccessfulCreate  39s   replicaset-controller  Created pod: frontend-6rw4c
          Normal  SuccessfulCreate  39s   replicaset-controller  Created pod: frontend-f7prw

    Managing ReplicaSets:
    --------------------
    > Deleting ReplicaSet and its pods:
      # kubectl delete <commad>, this will scale down the replicaset to zero. Then delete each pod and then deletes teh replicaset itself.

      Example:
       # kubectl delete rs/frontend
       # kubectl get pods
        > This will delete the pods managed by the replicaset and replicaSet itself.
    
    > Deleting Just replicaSet but not the pods:
      # kubectl delete --cascade=false <command>
       As the replicaset is deleted, the pods will be running by its own and vulnerable. It needs a new replicaset with similar Selector created to take over those pods. This is an example of loosely coupled  replicaSet. When   pods are being adopted the new pod templates are not updated to the pods. And it requires a rolling update. 
      
      Example:
       # kubectl delete rs/frontend --cascade=false
       # kubectl get rs 
       # kubectl get pods 
        We could see that the pods are still running but without the replicaSet.
       
    > Isolating Pods from ReplicaSet. 
      When change the labels on a pod, it no longer satisfy Selector in a replicaSet, and results in pod isolation. Its to be noted that when a node is isolated, the replicaSet will provision a new node to match the replicas. 

      Example:
       # kubectl get pods -l tier=frontend
       # kubectl label pods frontend-24584 tier-
         This will make sure a pod label called tier is removed. 
       # kubectl get pods -l tier=frontend
         List all pods again with the label, you should see the difference. 
       # kubectl get pods
         Tells that a new pod probably spun up as the replicationSet had one missing pod, which has the label changed. 
       # kubectl delete rs/frontend
       # kubectl get pods -l tier=frontend
         - Test by deleting the replicationSet, could see that isolated pod is still running.

    > Scaling a ReplicaSet:
      All we need to do is update the replicas number in the ReplicaSet. 
      # kubectl edit rs frontend
      #  kubectl scale --replicas=2 rs/frontend
    
    > Autoscaling ReplicaSet:
     HPA Horizontal Pod AutoScaler Target is used for Auto Scaling the replicaSet. 
     Horizontal Pod Autoscalers (HPA)
       - This is an API resource available with in the autoscaling API group in kubernetes. 
       - Allows to specify a target, which will be a replicaSet and the min and max number of replicas along with the scaling policy. 
       - HPA can be used to scall different controllers like ReplicationControllers, Deployments, ReplicaSets
       - Policy for scaling can be of CPU Utilization, custom-metrics or application provided metrics. 
       - HPA cannot work with nonscaling objects such as DaemonSets.
       - Thrashing is always a risk with autoscaling, where the pods can be continuosly scaled up and down. To address this issue, need to make sure a cooldown period is applied for scaling up or down. 
        This is configured
          -horizontal-pod-austocaler-downscale-delay with default value of 5 mins and 
          -horizontal-pod-austocaler-upscale-delay with default value of 3 mins. 
    
      One can create a caling HPA with below command. 
      # kubectl autoscale rs frontend --min=5 --max=10 --cpu-percent=50


  =======================
  Replication Controls: 
  =======================
    Will ensure the desired state of application. Will ensure always the number of declared pods are always running. 
    - The functionality for replicaSet + Deployments were intially managed by Replication Controllers. 
    - Replcation Controllers are replaced by ReplicaSet and Deployments.
    - This is outdated now, however they are replicaSet with functionality of rolling update. 
    
    Creating ReplicationController:

    Example: ReplicationController.yaml 
      apiVersion: v1
      kind: ReplicationController
      metadata:
        name: nginx
      spec:
        replicas: 3
        selector:
          app: nginx
        template:
          metadata:
            name: nginx
            labels:
              app: nginx
          spec:
            containers:
            - name: nginx
              image: nginx
              ports:
              - containerPort: 80
     
     #  kubectl describe rc/nginx
      Name:         nginx
      Namespace:    default
      Selector:     app=nginx
      Labels:       app=nginx
      Annotations:  <none>
      Replicas:     3 current / 3 desired
      Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
      Pod Template:
        Labels:  app=nginx
        Containers:
        nginx:
          Image:        nginx
          Port:         80/TCP
          Host Port:    0/TCP
          Environment:  <none>
          Mounts:       <none>
        Volumes:        <none>
      Events:
        Type    Reason            Age   From                    Message
        ----    ------            ----  ----                    -------
        Normal  SuccessfulCreate  15s   replication-controller  Created pod: nginx-9jcpj
        Normal  SuccessfulCreate  15s   replication-controller  Created pod: nginx-4xbcm
        Normal  SuccessfulCreate  15s   replication-controller  Created pod: nginx-llk58

     A pod created with replicationcontroller will be having below details. 
    # kubectl describe pod/nginx-84prf
    Controlled By:      ReplicationController/nginx

    Deleting ReplicationController:
    # kubectl delete rc/nginx

   
    Note: Same like in replicationSets the pods can be decoupled by deleting the labels.  
    Delete ReplicationController without deleting the pods.
    # kubectl delete rc/nginx --cascade=false 

    Isolate the pods. 
    1. change label of pods. 
    # kubectl label pods/nginx-4xbcm app-
    2. delete the ReplicationController
    # kubectl delete rc/nginx 
     This will leave the isolated pod still running. 


  =======================
  Deployment:
  =======================
  Overview:
  --------
   - Deployment encapsulates a ReplicaSet along with Versioning, Fast rollback and advanced deployments. 
   - Achieves versioning, rollbacks and blue green deployments. 
   - ReplicaSet Associate with the Deployment.
    - When a change is made to the deployment, Example: by updating the container version in Deployment template. 
      > A new ReplicaSet is created with a new set of Pods with new containers are running. 
      > Old ReplicaSet continues to exist 
      > Pods running on Older ReplicaSet is taken down gradually to Zero. 
   - Ease of Rollback:
    - Every change to the template is tracked by K8s 
    - Every changes are marked with a version. 
    - This helps in rollback to any previous version.
   - This infact provides, Versioning, Instant Rollback, Rolling deployment, Blue-Green deployment, and Canary deployment.

  Use case with Deployment:
  ------------------------
   - Used to rollout a ReplicaSet. 
   - Update the state of an existing deployment. Example: Update the pod template. 
   - Rollback to earlier version. 
   - Scale up the replication 
   - Paue/Resume deployments in mid-way. 
   - Check the status of the deployment. 
   - Cleanup older replicaSet which are not needed any more. 
  
  Create Deployment:
  ----------------- 
   - Selector matches the pods label to manage the pods. Selectors should be unique to a Deployment.
   - Stratergy: Defines how the old pods are replaced. 
     - spec.stratergy.type==Recreate 
       This will recreate the pods by killing existing pods before creating new ones 
     - spec.stratergy.type==RollingUpdate
       Change is applied in controlled fashion. 
       - spec.strategy.rollingUpdate.maxUnavailable
          Number of pods that can be unavailable during RollingUpdate. it can be number or percentage. 
       - spec.strategy.rollingUpdate.maxSurge
          Maximum number of pods that can be created over and above the number of ReplicationSet.
   - progressDeadlineSeconds: Time until the deployment is marked as failed 
   - minReadySeconds: Min number of Sec to be kept in ready state before marking available. 
   
  Creating the Deployment imperatively:
  ------------------------------------
   # kubectl run example --image=nginx 
   # kubectl describe pod example-7c8d5d7946-fdrch
        Labels:             pod-template-hash=3748183502
                            run=example
        Annotations:        <none>
        Status:             Running
        IP:                 192.168.77.127
        Controlled By:      ReplicaSet/example-7c8d5d7946   

   # kubectl delete deploy/example
     Deletes the deployment
   
  Creating the Deployment Declaratively:
  -------------------------------------
  # cat deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: nginx-deployment
      labels:
        app: nginx
    spec:
      replicas: 3
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
            image: nginx:1.7.9
            ports:
            - containerPort: 80
  
  # kubectl apply -f deployment.yaml 

  Rolling Back Deployment: 
  -----------------------
   - Ease of rollback is feature in deployment.
   - Every change to deployment template is tracked. 
   - A new revision is only created if the deployment pod template is changed. 
     Update label, container image etc. 
   - Scaling a deployment will not create a new revision. 
   - Rolling updates creates a new replicaSet. Started spinning up pods in new replicaSet and once everything is up it will delete the pods from older replicaSet. 
   - Seen that the replicaSet is still saved for future reference for rolling back. As it adds pods on the version of replicaSet where the deployment was asked to rollback and deletes the pods on current running replicaSet.

  
   - Rolling to a revision can be done using below command. 
   # kubectl rollout undo deployment/nginx-deployment
   
   - Once could rollout to a revision number using below command. 
   # kubectl rollout undo deployment/nginx-deployment --to-revision=2 

   - A list of revision for the deployment can be found using below command. 
   # kubectl rollout history deployment/nginx-deployment

   - To record the command that changed the revision, --record switch can be used. 
   # kubectl apply -f deployment.yaml --record
    - or -
   # kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record

   # kubectl rollout history deployment/nginx-deployment
      deployment.extensions/nginx-deployment 
      REVISION  CHANGE-CAUSE
      0         <none>
      1         kubectl apply --filename=deployment.yaml --record=true 
      7         kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record=true
  
  Performing a Rollback. 
  # kubectl rollout history deployment/nginx-deployment
      deployment.extensions/nginx-deployment 
      REVISION  CHANGE-CAUSE
      0         <none>
      3         kubectl apply --filename=deployment.yaml --record=true
      4         kubectl apply --filename=deployment.yaml --record=true

  Find more details on a revision 
  # kubectl rollout history deployment/nginx-deployment  --revision=3
      deployment.extensions/nginx-deployment with revision #3
      Pod Template:
        Labels:	app=nginx
        pod-template-hash=2315082692
        Annotations:	kubernetes.io/change-cause: kubectl apply --filename=deployment.yaml --record=true
        Containers:
        nginx:
          Image:	nginx:1.7.9
          Port:	80/TCP
          Host Port:	0/TCP
          Environment:	<none>
          Mounts:	<none>
        Volumes:	<none>

  # kubectl rollout undo deployment/nginx-deployment --to-revision=3
      deployment.extensions/nginx-deployment rolled back
  # kubectl get pods
      NAME                                READY   STATUS        RESTARTS   AGE
      nginx-deployment-66dbc4555c-8b956   0/1     Terminating   0          1m
      nginx-deployment-66dbc4555c-kg9v4   0/1     Terminating   0          1m
      nginx-deployment-66dbc4555c-kr2f4   0/1     Terminating   0          1m
      nginx-deployment-67594d6bf6-2mg78   1/1     Running       0          2s
      nginx-deployment-67594d6bf6-tr6k2   1/1     Running       0          3s
      nginx-deployment-67594d6bf6-xnn4k   1/1     Running       0          5s

  Pausing/Resuming Deployment:
  ---------------------------
  - This can be performed imperatively or Declaratively. 
  To Pause a deployment
  # kubectl rollout pause deployment/nginx-deployment

  To Resume a deployment
  # kubectl rollout resume deployment/nginx-deployment

  - When a deployment is paused, changes to the pod template will not take effect until its resumed. 
  - Cannot rollback paused deployment. It need to resume first. 


  Clean-Up Policy in Deployment:
  -----------------------------
  - When a new revision is created there is an associated ReplicaSet is created. 
  - .spec.revisionHistoryLimit can control how many such revision can be kept. 
  - Setting up .spec.revisionHistoryLimit to zero makes sure no revision is kept. 

  Scaling the Deployment:
  ----------------------
   - Declaratively scale the deployment:
   Update below spec of Deployment
   spec:
     replicas: 3
   - imperatively scale the deployment:
   # kubectl scale deployments nginx-deployment --replicas=1

  Rolling Deployments:
  -------------------
  - Deployment, updates are pushed incremently. 

  # kubectl scale deployments/nginx-deployment --replicas=10
  # kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 --record
   We could see that new containers have been creating one by one. 

  Deploying an Application, Rolling Updates, and Rollbacks Example2:
  -----------------------------------------------------------------
    # cat kubeserve-deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kubeserve
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: kubeserve
      template:
        metadata:
          name: kubeserve
          labels:
            app: kubeserve
        spec:
          containers:
          - image: linuxacademycontent/kubeserve:v1
            name: app

    Create a deployment with a record (for rollbacks):
    # kubectl create -f kubeserve-deployment.yaml --record

    Check the status of the rollout:
    # kubectl rollout status deployments kubeserve

    View the ReplicaSets in your cluster:
    # kubectl get replicasets

    Scale up your deployment by adding more replicas:
    # kubectl scale deployment kubeserve --replicas=5

    Expose the deployment and provide it a service:
    # kubectl expose deployment kubeserve --port 80 --target-port 80 --type NodePort

    Set the minReadySeconds attribute to your deployment:
    # kubectl patch deployment kubeserve -p '{"spec": {"minReadySeconds": 10}}'

    Use kubectl apply to update a deployment:
    # kubectl apply -f kubeserve-deployment.yaml

    Use kubectl replace to replace an existing deployment:
    # kubectl replace -f kubeserve-deployment.yaml

    Run this curl look while the update happens:
    # while true; do curl http://10.105.31.119; done

    Perform the rolling update:
    # kubectl set image deployments/kubeserve app=linuxacademycontent/kubeserve:v2 --v 6

    Describe a certain ReplicaSet:
    # kubectl describe replicasets kubeserve-[hash]

    Apply the rolling update to version 3 (buggy):
    # kubectl set image deployment kubeserve app=linuxacademycontent/kubeserve:v3

    Undo the rollout and roll back to the previous version:
    # kubectl rollout undo deployments kubeserve

    Look at the rollout history:
    # kubectl rollout history deployment kubeserve

    Roll back to a certain revision:
    # kubectl rollout undo deployment kubeserve --to-revision=2

    Pause the rollout in the middle of a rolling update (canary release):
    # kubectl rollout pause deployment kubeserve

    Resume the rollout after the rolling update looks good:
    # kubectl rollout resume deployment kubeserve

    > minReadySeconds specifies how long should the newly created pod should be ready before it is considered available. The rollout will not continue until the pod is available. This will make sure that there are no bad pods marked as available during a rollout. 
    > When a reeadinessProbe will add the pod to the network. Means if the readinessProbe is success it signals the pods to access the traffic. 

    # cat buggyreadinessprone.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: kubeserve
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: kubeserve
      minReadySeconds: 10
      strategy:
        rollingUpdate:
          maxSurge: 1
          maxUnavailable: 0
        type: RollingUpdate
      template:
        metadata:
          name: kubeserve
          labels:
            app: kubeserve
        spec:
          containers:
          - image: linuxacademycontent/kubeserve:v3
            name: app
            readinessProbe:
              periodSeconds: 1
              httpGet:
                path: /
                port: 80

    Apply the readiness probe:
    # kubectl apply -f kubeserve-deployment-readiness.yaml

    View the rollout status:
    # kubectl rollout status deployment kubeserve

    Describe deployment:
    # kubectl describe deployment




  =======================
   StatefulSet: 
  =======================
   - Similar to deployment manages pods. 
   - Difference is that it manages a sticky identity with each pod. 
   - Pods are not interchangable and had identifier which maintans across any rescheduling. 
   - Orderly perform a graceful deployment and scaling .
   - orderly perform a graceful deletion and termination.
   - Oderly perform automate rolling updates. 
   - Used if needed a stable network identifier or persistent storage is required.
   - Deleting and scaling a statefilSet will not delete the volumes associated with it. 
   - Pods are created sequencially 0...n..n+1, also termination in reverse order. 

   Example: StatefulSet.yaml
      apiVersion: apps/v1
      kind: StatefulSet
      metadata:
        name: web
      spec:
        serviceName: "nginx"
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
              image: k8s.gcr.io/nginx-slim:0.8
              ports:
              - containerPort: 80
                name: web

   - One should see the pods are being created in sequence. From zero to one. 
   - Any delete operation will also be in sequence From one to Zero. 
   
  Creating a Self-Healing Application
  -----------------------------------
  > What if need to have the pods unique. When one dies it is relaced with the same hostname 
  > The service should be headless so the traffic reaches the same pod. 
  > Each pod in StatefulSet gets its own storage. 
    The YAML for a StatefulSet:
    # cat statefulset.yaml 
    apiVersion: apps/v1
    kind: StatefulSet
    metadata:
      name: web
    spec:
      serviceName: "nginx"
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
            image: nginx
            ports:
            - containerPort: 80
              name: web
            volumeMounts:
            - name: www
              mountPath: /usr/share/nginx/html
      volumeClaimTemplates:
      - metadata:
          name: www
        spec:
          accessModes: [ "ReadWriteOnce" ]
          resources:
            requests:
              storage: 1Gi

    Create the StatefulSet:
    # kubectl apply -f statefulset.yaml

    View all StatefulSets in the cluster:
    # kubectl get statefulsets

    Describe the StatefulSets:
    # kubectl describe statefulsets



  =======================
    DaemonSet: 
  =======================
  - Make sure that pods are running on all or specific subset of nodes that satisfy a condition.
  - If a node is added a pod will get added to the node. 
  - If a node is deleted, the corresponding pods are garbage collected. 
  - Common usecase:
   - Running a cluster storage daemon, 
   - Log collection on each node ,
   - Node monitoring tools like prometheus 
  - Alternatives to DaemonSet:
  - This is used to run pods without an help of scheduler. Makes sure that it runs pods on each node or satisfying a condition.
  - DaemonSet ignores the taints on a node.

  Create DaemonSet
  ----------------
    Behavior of DaemonSet.
  Find the DaemonSet pods that exist in your kubeadm cluster:
  # kubectl get pods -n kube-system -o wide

  Delete a DaemonSet pod and see what happens:
  # kubectl delete pods [pod_name] -n kube-system
  - This will recreate the pod once again.

  Create DaemonSet pods on nodes based on the tags.
  Give the node a label to signify it has SSD:
  # kubectl label node[node_name] disk=ssd

  The YAML for a DaemonSet:
  # cat daemset.yaml 
  apiVersion: apps/v1beta2
  kind: DaemonSet
  metadata:
    name: ssd-monitor
  spec:
    selector:
      matchLabels:
        app: ssd-monitor
    template:
      metadata:
        labels:
          app: ssd-monitor
      spec:
        nodeSelector:
          disk: ssd
        containers:
        - name: main
          image: linuxacademycontent/ssd-monitor

  Create a DaemonSet from a YAML spec:
  # kubectl create -f daemset.yaml

  Label another node to specify it has SSD:
  # kubectl label node ip-192-168-223-238 disk=ssd

  View the DaemonSet pods that have been deployed:
  # kubectl get pods -o wide

  Remove the label from a node and watch the DaemonSet pod terminate:
  # kubectl label node ip-192-168-223-238 disk-

  Change the label on a node to change it to spinning disk:
  # kubectl label node ip-192-168-223-238 disk=hdd --overwrite

  Pick the label to choose for your DaemonSet:
  # ubectl get nodes ip-192-168-223-238 --show-labels



  =======================
    JOBS 
  =======================
   - Pods are started and executes a task and get terminates. 
   - Will create one or more pods, with the specification .
   - JOB will track the number of pods that are completed, and when the completion count mets it mark the job as completed. 
   - Deleting a Job will cleanup the job that is created. 
   Types:
    - Non Parallel jobs: 
        Make sure one pod runs through successful completion. 
    - Parallel Jobs with fixed completion count: 
        Specifies how many pods needs to be completed. 
    - Parallel jobs with work queue: 
        Here the pods are continued with operation until the work queue is completed. 

  Example:
  # cat job.yaml 
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: pi
    spec:
      template:
        spec:
          containers:
          - name: pi
            image: perl
            command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
          restartPolicy: Never
      backoffLimit: 4 

 #  Now to see the result first find the pods 
 # kubectl get pods --selector=job-name=pi --output=jsonpath={.items..metadata.name}
 Get the logs: As the command is executed the result will be posted to STDOUT which can be viewed with logs. 
 # kubectl logs pi-87vg4 

Create a job that runs 20 times, 5 containers at a time and prints 'Hello Parallel World'
-----------------------------------------------------------------------------------------
apiVersion: batch/v1
kind: Job
metadata:
  name: process-item-apple
  labels:
    jobgroup: jobexample
spec:
  completions: 20
  parallelism: 5
  template:
    metadata:
      name: jobexample
      labels:
        jobgroup: jobexample
    spec:
      containers:
      - name: c
        image: busybox
        command: ["sh", "-c", "echo Hello Parallel World && sleep 1"]
      restartPolicy: Never

      # ka jobs.yaml 
      # for p in $(kubectl get pods -l jobgroup=jobexample -o name); do   kubectl logs $p; done| wc -l

  =======================
    CRON JOBS 
  =======================
   - Manages the time based job. 
  

  =======================
  Labels and selectors: 
  =======================
   Key Value pairs or tags  for identification. Can be used as a reference for applying configuration collectively.  
   Example: kubectl get po -l app=24
   - Labels can be used to organize and bring certain objects together. 
   - can be added at creation or at later time. 
   Example:
   # kubectl get pods -l tier=frontend
    - lists all the pods with the label tier = frontend. 
   # kubectl label pods/frontend-24584 tier2=frontend2
     - labels a pod with tier2 = frontend2 
   # kubectl label pods/frontend-cq2cd tier-
     Removes the label from the pod. 

  =======================
   Annotations
  =======================
  - Metadata associated with K8 Objects, they are very similar to Label. 
  - Labels tend to be used for identification and group of objects to satify certain condition. But Annotations are not used for identification, but used to store metadata. 
  - They can be larger than labels, can have special characters. 


=======================
Addons
=======================
 Additional componentes that can be installed to allow Additional functionality in Kube Cluster. Implemented as Pods and Services.  
> DNS 
> Web UI 
> Container Resource monitoring.
> Cluster-Level logging system 
 More Addons: https://kubernetes.io/docs/concepts/cluster-administration/addons/


=======================
Networking in K8s 
=======================
 Docker networking is via Bridge and port mapping of the Instance network. 

 On Kubernetes: Chooses a model, where pods can be treated as much as VM or Bare metal server with the IP. Having a Flat network without a NAT. Pods gettting same IP range as the worker node. 
 - Kubernetes applies the IP address to a POD rather than a single container. 
 - Kubernetes creates a network namespace and runs pods in that. The Node network and localhost is shared with the pods.  
 - As each pods gets the IP with are real and routable, the same IP is used to discover by pods internally or externally.
 
 The Networking requirements for K8s:
  - All containers should communicate with all other containers without NAT. 
  - All nodes can communicate with all containers and viceversa without NAT. 
  - IP that a container sees itself as is the same IP that others see it as.

 POD to POD in same node:
 ----------------------- 
 > Each POD has IP and is received from the virtual ethernet intenrface. 
  The network does flows as below. 
 > POD1 IPaddress on eth0 ---> veth111abc11(Virtual ethernet) ---> BRIDGE Network ---> veth222abc222 -> eth0 on POD2
  
Digging the network interface of the POD.  
  One can see the virthal ethernet adaptor from the nodes using 
 # ifconfig
  Also the Brigde network layout using below command. 
 # brctl show
    bridge name	      bridge id		      STP enabled	    interfaces
    cni0		          8000.ae6c2e1d8ecb	  no		        veth496f3bac
                                                        veth761c156c
                                                        veth7fc21199
                                                        vethc3c4ca50
                                                        vethe2a22225
 
 > docker ps -a from the nodes should list all the running containers. 
 Example: 
  # docker ps -a 
  94d18bddcfef        nginx                  "nginx -g 'daemon of…"   16 minutes ago      Up 16 minutes       k8s_nginx_nginx-7cdbd8cdc9-2gp29_default_a6186356-a50d-11e9-8f51-0ad57078477c_0
  73b23d0ce11a        k8s.gcr.io/pause:3.1   "/pause"                 16 minutes ago      Up 16 minutes       k8s_POD_nginx-7cdbd8cdc9-wk8gn_default_a618925f-a50d-11e9-8f51-0ad57078477c_0

  First one here is the nginx container which is the container of a Pod and the second one is Pause container. Pause containers are to pretains to the nginx container for the purpose of holding it to the network namespace. 

  Lets find the network namespace for the nginx container to figure out the virtual network interface on the node, which this container is connected to. 
  # docker inspect --format '{{ .State.Pid }}' 94d18bddcfef
   29951
   - This gives the process ID. 

   Use Name Space enter command to find the network interface used by this process ID. 
  # # nsenter  -t 29951 -n ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
          valid_lft forever preferred_lft forever
    3: eth0@if18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 8951 qdisc noqueue state UP group default 
        link/ether 02:38:23:b1:22:ab brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet 10.244.1.36/24 scope global eth0
          valid_lft forever preferred_lft forever

          - From this output, eth0@if18 should the interface and 'if18' means that it is the 18th network interface on the host.   

 POD to POD in different node: 
 ----------------------------
  > Communication happens from a POD on node1 to Pod on node2 is communicated as below. 

  POD1 eth0 ---> veth111abc11(Virtual ethernet) ---> BRIDGE Network Node 1 ---> eth0 of Node1 ---> eth0 Node2 ---> BRIDGE Network Node 2---> veth222abc222 -> eth0 on POD2

  > In this communication as packet leaves the Bridge network the network packet is being changed. It is NATed to the eth0 address of the Node1. Also before reaching on Node Bridge network 2, the packets are decapsulated and send to the pod via virtial interface. This node to node communication is done throgh container network interface or called CNI. 
  > This can be configured with manual L3 router, but as new pods being provisioned, it needs to be automated hence CNI comes into picture. 

 Container Network Interface (CNI):
 ---------------------------------
 > Aids in the communication from Node to node. Its the networking overlay. Allowing us to build a tunnel between nodes so that pods can communicate. 
 > It encapsulates the packets and change the source and destination. 
 > Flannel, calico are the examples. 
 > CNI installs the network agent which ties the CNI to the network interface. Kubelet should know if CNI plugin is used. This is done by notifying the kubeadm to build cluster with the option --pod-network-cidr=10.244.0.0/16
 > Takes care of IPAM Ip address management. 
  

=======================
Service
=======================
  The way to expose the collection of pods Internally or externally. Service is responsible for the Discovery of the services running. Abstraction defines the lofical set of pods and policy by which you access them, also called as micro services.
 - If a pod gets deleted, the pod by default is created with a different IP. So the client needs to be addressed with the new change in the pod IP. These porblems are solved with Kubernetes service.
 - Pods IP address is ephemeral and may get changed at anytime.
 - A kubernetes service provides constant IP and ports as an entry point for group of pods. Kubernetes creates a specialized range of IP address called ClusterIP and provides to the service. 
 - NodePort service type, exposes the service by mapping to a port on every node of a cluster. 
 - Loadbalancer Service type, Pods runs behind the Loadbalancer.
 - Ingress Controllers. 
 - Services in Kubernetes are allowing to expose the pods. 
 - Service object uses the Label selectors to choose the pods. 
 - Endpoint objects keeps track of the current state of pods internally. 
 - Unlike in ECS Service, Kubernetes service is only grouping pods together and exposes it.. The number of pods to be running is the responsible of ReplicationController. 
 - DNS SRV allows to lookup port number of a POD. SRV records are created to all type of services. 
   - Record will have the form of : port-name.port-protocol.my-svc.my-namepace.svc.cluster.local. This is going to resolve the port number and the CNAME. 
 - Simplifies locating critical applications. Service provides a single virtual interface and evenly distributed among all the pods. 
Securing Service:
----------------
 - Make use of TLS Cert. 




 - Service Types:
   - ClusterIP 
     The default service type, Having an IP address in a range that only existis inside the kubernetes cluster. It cannt be accessed from outside. Only accessible with in the Custer. This is the default service type. 
     - Service is available only from the Node Cluster. Example: CoreDNS
     - DNS resolves to myService.myNamespace.svc.cluster.local 

   - NodePort 
     Every single worker node will listen on a port and redirects the traffic reaching the worker node to a pod. Allowing to access from outside. 
     Here, the service is exposed on each node on a static port. Any client hitting the any node IP on the node port, the traffic will be routed by kube-proxy to the correct pod. 
     - When traffic hits Node IP + Node Port -> Kubeproxy will forward the traffic to corresponding Cluster IP + NodePort. Important to note that whenever there is a NodePort service created a corresponding ClusterIP will get created within the same service.

   - Loadbalancer 
     CloudController requests the Loadbalancer using the Cluster role assigned and all the Worker nodes are added to the Loadbalancer. The Target groups are registered to the port of the application running on the worker Node. Its no difference that the NodePort service type. 
     - When a Loadbalancer service is created a NodePort and Cluster IP is created under the hood. 
     - Traffic hits external LB -> which forwards to the NodePort -> Kubeproxy forwards to ClusterIP -> To Backend Pod.
      Two ways. 
       Service type can be LoadBalancer: This is managed by the CloudController manager. 
       Service type can be Ingress: Allows to create an entry point using Customized LoadBalancers eg nginx, AWS ALB. Controlled by ingress master controller. 

   - ExternalName 
     Using ExternalName service - One could create service called Database and add the URL of RDS, so anyone want to connect to the external database.

  Example:
   # replicaset: 
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: nginxreplica
      labels:
        app: nginx
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
            image: nginx
            ports:
              - containerPort: 80
                protocol: TCP

   # nginxsvc.yaml 
      kind: Service
      apiVersion: v1
      metadata:
        name: nginxsvc 
        labels:
          app: nginxsvc
      spec:
        ports:
        - port: 80
          protocol: TCP
        selector:
          app: nginx
        type: NodePort

   Example: Create service using expose command on depoyment. 

    # kubectl run hellow --replicas=2 --labels="run=lb" --image=gcr.io/google-samples/node-hello:1.0 --port=8080
    # kubectl expose deployment hellow --type="LoadBalancer" --name="hellowsvc"
    # kubectl expose deployment kubeserve2 --port 80 --target-port 8080 --type LoadBalancer
  
  Example 2:
   How service is being routed with IPTables. 

    # k describe svc/nginxsvc
      Name:                     nginxsvc
      Namespace:                default
      Labels:                   app=nginxsvc
      Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                                  {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"nginxsvc"},"name":"nginxsvc","namespace":"default"},"spe...
      Selector:                 run=nginx
      Type:                     NodePort
      IP:                       10.101.195.97
      Port:                     <unset>  80/TCP
      TargetPort:               80/TCP
      NodePort:                 <unset>  32335/TCP
      Endpoints:                10.244.1.35:80,10.244.1.36:80,10.244.1.37:80 + 7 more...
      Session Affinity:         None
      External Traffic Policy:  Cluster
      Events:                   <none>
      
      - In above example the service is running on NodePort 32335. So kube-proxy will make sure that the IPtables are updated as below to route the traffic to/from the instance. 
    
    # iptables-save | grep -i kube | grep -i nginx
      -A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginxsvc:" -m tcp --dport 32335 -j KUBE-MARK-MASQ
      -A KUBE-NODEPORTS -p tcp -m comment --comment "default/nginxsvc:" -m tcp --dport 32335 -j KUBE-SVC-W5BISDWII3IMXNQQ
      -A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.101.195.97/32 -p tcp -m comment --comment "default/nginxsvc: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
      -A KUBE-SERVICES -d 10.101.195.97/32 -p tcp -m comment --comment "default/nginxsvc: cluster IP" -m tcp --dport 80 -j KUBE-SVC-W5BISDWII3IMXNQQ

  EndPoints:
  ---------
 * Networking in Docker:
  - Docker uses the Host private networking, which communicates each other within a node. 
  - Ports must be allocated on node IPs. 
  - Container needs to be allocated with ports. 
  - it keeps the cache of IP addresses of the nodes.

 * Networking in Kubeternets:
  - Pods can communicate each other, This is made possible with Pods with provate IP address. 
  - Allows inter pod communication independent of nodes. 
  - Container within a pod can access with localhost. 
  - Container accross pods should use POD IP address. 

  - Endpoints relays the traffic reaching to the service to backend. 
  - Dynamic list is updated as pods are created/deleted.

 Service without a Selector:
 --------------------------
 - If backend is not going to be a logical set of pod, then you may need to create a service without selector. In this case need to manually map the specific service with a specific IP. 
 

 MultiPort Service: 
 -----------------
  - More than one port can be exposed on a service.  Example, 80 and 443. Each ports should be named. 


Headless Services
-----------------
 - Headless services are type of service, created without the ClusterIP 
 - Used if not required a single/Loadbalacned IP. 
 - This is created by specifying ClusterIP: none at the spec. 
 - Kube Proxy will not resolve service to the Headless service. 
 - Service will have a DNS record and resolve to a set of IPs of all of the pods that is assigned to the service. 

  # cat  headless.yaml 
    apiVersion: v1
    kind: Service
    metadata:
      name: kube-headless
    spec:
      clusterIP: None
      ports:
      - port: 80
        targetPort: 8080
      selector:
        run: nginx

  $ kubectl apply -f headless.yaml 
  service/kube-headless created
  # kubectl get svc
  NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
  kube-headless   ClusterIP   None            <none>        80/TCP         6s

 Headless service returns the POD IP address directly.
  $ kubectl exec -it busybox -- nslookup kube-headless
  Server:    10.96.0.10
  Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

  Name:      kube-headless
  Address 1: 10.244.2.52 10-244-2-52.kube-headless.default.svc.cluster.local
  Address 2: 10.244.2.54 10-244-2-54.kube-headless.default.svc.cluster.local
  Address 3: 10.244.2.47 10-244-2-47.kube-headless.default.svc.cluster.local
  Address 4: 10.244.2.55 10-244-2-55.kube-headless.default.svc.cluster.local
  Address 5: 10.244.2.46 10-244-2-46.kube-headless.default.svc.cluster.local
  Address 6: 10.244.2.49 10-244-2-49.kube-headless.default.svc.cluster.local
  Address 7: 10.244.2.48 10-244-2-48.kube-headless.default.svc.cluster.local
  Address 8: 10.244.2.53 10-244-2-53.kube-headless.default.svc.cluster.local
  Address 9: 10.244.2.51 10-244-2-51.kube-headless.default.svc.cluster.local
  Address 10: 10.244.2.45 10-244-2-45.kube-headless.default.svc.cluster.local


YAML spec for a custom DNS pod:
------------------------------
# cat customdns.yaml 
  apiVersion: v1
  kind: Pod
  metadata:
    namespace: default
    name: dns-example
  spec:
    containers:
      - name: test
        image: nginx
    dnsPolicy: "None"
    dnsConfig:
      nameservers:
        - 8.8.8.8
      searches:
        - ns1.svc.cluster.local
        - my.dns.search.suffix
      options:
        - name: ndots
          value: "2"
        - name: edns0

  $ kubectl exec -ti custdnssybox -- cat /etc/resolv.conf 
  nameserver 8.8.8.8
  search ns1.svc.cluster.local my.dns.search.suffix
  options ndots:2 edns0

  # kubectl exec -it custdnssybox -- nslookup kube-dns.kube-system.svc.cluster.local
  Server:    8.8.8.8
  Address 1: 8.8.8.8 dns.google

  nslookup: can't resolve 'kube-dns.kube-system.svc.cluster.local'
  command terminated with exit code 1
   Fails the local query. 
  
  $ kubectl exec -it custdnssybox -- nslookup google.com
  Server:    8.8.8.8
  Address 1: 8.8.8.8 dns.google

  Name:      google.com
  Address 1: 2607:f8b0:4004:814::200e iad30s24-in-x0e.1e100.net
  Address 2: 172.217.164.142 iad30s24-in-f14.1e100.net


 ExtenelIPs
 ---------
  - Specifying externalIPs in the service.spec.externalIPs will allow to route traffic arriving on this IP to internal pods. 


 DNS on Pods:
 -----------
  - PODs dns policies informs how the pod can access rest of the world. 
    - spec.dnsPolicy = Default : Inherits nodes Name resolution. 
    - spec.dnsPolicy = ClusterFirst: Records that are not matching the DNS suffix settings, Forwards to upstream name server inherited from the nodes. 
    - spec.dnsPolicy = ClusterFirstWithHostNet
    - spec.dnsPolicy = None: Allow policy to ignone all DNS settings from K8 environment. DNS settings taken from the DNS config field in pods spec/ 


=======================
 Ingress Rules and LoadBalancer:
=======================
LoadBalancer:
> LoadBalancer is an extension of NodePort type.
 - LoadBalancer redirects traffic to all nodes and node ports.
 - Client reaches LoadBalancer IP address. 
 $ k get svc
    NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        47h
    nginxsvc     NodePort    10.101.195.97   <none>        80:32335/TCP   47h
 - Two issues is that with above service external clients cannot access the service. 
 - CloudController manager provisons the LoadBalancer. 
 - LoadBalancer are not pod aware. For example, if a node has 2 or more pod and another node with single pod, the Traffic between all the Pods are not evenly distributed by the LoadBalancer. Hence kubernetes uses IPtables to fix this. In other  words K8 uses Iptables to route the traffic to pods from the LoadBalancer. This extra hop creates Latency.

 This can be turned off by annotating the Service Older version of K8?? In new version it has to be adding service.spec.externalTrafficPolicy ??

 # kubectl describe svc/nginxsvc 
 # kubectl annotate service nginxsvc externalTrafficPolicy=Local

 Below you could find the deprecated Beta annotations used to enable this feature prior to its stable version. Newer Kubernetes versions may stop supporting these after v1.7. Please update existing applications to use the fields directly.
    service.beta.kubernetes.io/external-traffic annotation <-> service.spec.externalTrafficPolicy field
    service.beta.kubernetes.io/healthcheck-nodeport annotation <-> service.spec.healthCheckNodePort field

 Ingress:
  > Using ingress can access multiple services running inside. Example alb ingress controller. 
  > Ingress app.example.com/app1 app.example.com/app2 
  > both Ingress controller and ingress are created to get this done. 

 Example: 
      apiVersion: extensions/v1beta1
      kind: Ingress
      metadata:
        name: service-ingress
      spec:
        rules:
        - host: kubeserve.example.com
          http:
            paths:
            - backend:
                serviceName: kubeserve2
                servicePort: 80
        - host: app.example.com
          http:
            paths:
            - backend:
                serviceName: nginx
                servicePort: 80
        - http:
            paths:
            - backend:
                serviceName: httpd
                servicePort: 80

=======================
Service Discovery
=======================
 - Refers to a manner in which one pod/container can access the other services within the pods.
 - Environment Variables:
  All pods created after the service will contain env variable pointing to the service Port and IP. Pods can make use of that to connect. 
 - DNS 
  Requires KubeDNS cluster addons which manages the service IP / hostname resolution.
 
  Modes of service discovery.
  - DNS.
    - This is done by addon kube-dns. 
    - Each time a service object is created, a DNS record is created automatically by the dns addon. 
    - for example, the service nginxsvc will be resolvable once its up from a pod with the service name. Condition is that the pod should be in the same service.  
    - Pods in other service should be able to resolve the name by ServiceName.namespaceName. 
    - DNS lookup will return the ClusterIP of service. 
    - DNS SRV allows to discover which port is opened. 
    - DNS over TLS is supported. 
    - 
  - Environment variables.
    - Environment variables are static. 
    - Kubelete configures set of environment variables. 
    - They are not updated after pod creation and cannot be reliable. 

Kube DNS deployment as follow:
-----------------------------
    # kubectl get pods -n kube-system
    NAME                                        READY   STATUS    RESTARTS   AGE
    coredns-5c98db65d4-n8hq4                    1/1     Running   1          3h43m
    coredns-5c98db65d4-qvqvt                    1/1     Running   3          3h43m

    # k get deploy -n kube-system
    NAME      READY   UP-TO-DATE   AVAILABLE   AGE
    coredns   2/2     2            2           7d23h

    # k get services -n kube-system
    NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
    kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   7d23h
     - The name is still kube-dns, rather than coredns because, it is to provide backward compatabilty for tools that uses service name. 

 ON a POD DNS is configured as below: 
 -----------------------------------
  $ kubectl exec -it busybox -- cat /etc/resolv.conf
  nameserver 10.96.0.10
  search default.svc.cluster.local svc.cluster.local cluster.local ec2.internal
  options ndots:5

  $kubectl exec -it busybox -- nslookup kubernetes
  Server:    10.96.0.10
  Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
  Name:      kubernetes
  Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local

 - from the seatch field the domain name is added to the resolution. 

 Look up the DNS names of your pods:
  # kubectl exec -ti busybox -- nslookup 10-244-2-48.default.pod.cluster.local
  Server:    10.96.0.10
  Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

  Name:      10-244-2-48.default.pod.cluster.local
  Address 1: 10.244.2.48 10-244-2-48.nginxsvc.default.svc.cluster.localhost

 Lookup the DNS server itself from a pod:
    $ kubectl exec -it busybox -- nslookup kube-dns.kube-system.svc.cluster.local
    Server:    10.96.0.10
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

    Name:      kube-dns.kube-system.svc.cluster.local
    Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

 Check the logs of CoreDNS: 
  $ kubectl logs coredns-5c98db65d4-qvqvt -n kube-system
  .:53
  2019-07-19T01:29:39.806Z [INFO] CoreDNS-1.3.1
  2019-07-19T01:29:39.806Z [INFO] linux/amd64, go1.11.4, 6b56a9c
  CoreDNS-1.3.1
  linux/amd64, go1.11.4, 6b56a9c
  2019-07-19T01:29:39.806Z [INFO] plugin/reload: Running configuration MD5 = f65c4821c8a9b7b5eb30fa4fbc167769


=======================
  RBAC 
=======================
  Cluster Authentication and Authorization
  ----------------------------------------
   - RBAC is configured 
  - Used for identity and access management. 
  
   - Odentity can be of below types.
     - users.
     - groups.
     - service accounts. 

   - Types of Access Managements:
      Roles
    -------
      - Simple role governs permissions to a specific namespace. 
      - A role that represents a set of permissions. 
    Example:
    # cat role_podreader.yaml.
      apiVersion: rbac.authorization.k8s.io/v1
      kind: Role
      metadata:
        namespace: default
        name: pod-reader
      rules:
      - apiGroups: [""] # "" indicates the core API group
        resources: ["pods"]
        verbs: ["get", "watch", "list"]
  
    # this action will just create a role. This role is going to grant all pods running in default namespace with get, watch and list operations. 
    Now to need to MAP the role to a identity using Role binding. 

      Cluster Roles  
    ---------------
      - Applies permissions to entire cluster and all namespaces in a cluster. 
      - Used for sepcific operations such as cluster-scoped resources like nodes,. endpoints and for pods across all namespcaes etc. 

    # Example:
      apiVersion: rbac.authorization.k8s.io/v1
      kind: ClusterRole
      metadata:
        # "namespace" omitted since ClusterRoles are not namespaced
        name: secret-reader
      rules:
      - apiGroups: [""]
        resources: ["secrets"]
        verbs: ["get", "watch", "list"]  
    
    RoleBindings and ClusterRoleBindings:
    ------------------------------------
    - Once the Identity and Roles are in place, the association is achieved by Binding.
    - Both types of Bindinds can be applied to Either Role or ClusterRoles.
    - If a ClusterRole is binding using RoleBindings the permissions are applied only to the specific NameSpace defined in the RoleBinding.

    Two types of bindings. 
     - RoleBindings:
        Binds a Role to a specific Namespace.
      Example: Role binding a Normal Role 
        apiVersion: rbac.authorization.k8s.io/v1
        # This role binding allows "jane" to read pods in the "default" namespace.
        kind: RoleBinding
        metadata:
          name: read-pods
          namespace: default
        subjects:
        - kind: User
          name: jane # Name is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:
          kind: Role #this must be Role or ClusterRole
          name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
          apiGroup: rbac.authorization.k8s.io

      Example: Role Binding a Cluster Role 
        apiVersion: rbac.authorization.k8s.io/v1
        # This role binding allows "dave" to read secrets in the "development" namespace.
        kind: RoleBinding
        metadata:
          name: read-secrets
          namespace: development # This only grants permissions within the "development" namespace.
        subjects:
        - kind: User
          name: dave # Name is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:
          kind: ClusterRole
          name: secret-reader
          apiGroup: rbac.authorization.k8s.io

     - ClusterRoleBindings:
      Binds roles across entire cluster, and all namespaces in the cluster. 
      Example:
        apiVersion: rbac.authorization.k8s.io/v1
        # This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
        kind: ClusterRoleBinding
        metadata:
          name: read-secrets-global
        subjects:
        - kind: Group
          name: manager # Name is case sensitive
          apiGroup: rbac.authorization.k8s.io
        roleRef:
          kind: ClusterRole
          name: secret-reader
          apiGroup: rbac.authorization.k8s.io

 Example: 
  Pod accessing resources using the service account associated:
$ kubectl run test --image=chadmcrowell/kubectl-proxy -n myns
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/test created

$ kubectl exec -ti test-f57db4bfd-qfzl7  -n myns -- sh
/ # curl localhost:8000
curl: (7) Failed to connect to localhost port 8000: Connection refused
/ # curl localhost:8001
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {
    
  },
  "status": "Failure",
  "message": "forbidden: User \"system:serviceaccount:myns:default\" cannot get path \"/\"",
  "reason": "Forbidden",
  "details": {
    
  },
  "code": 403
 }/ # # 
  Here user cannot list or modify any resources in the serviceaccount. The token used to authenticate to the service account is "cat /var/run/secrets/kubernetes.io/serviceaccount/token "


    Example 2: 
    Access to list service for client. 
    ----------------------------------

    Create a new namespace:
    # kubectl create ns web

    # cat sample webservicelistrole.yaml 
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: web
      name: service-reader
    rules:
    - apiGroups: [""]
      verbs: ["get", "list"]
      resources: ["services"]

    Create a new role from that YAML file:
    # kubectl apply -f webservicelistrole.yaml 

    Create a RoleBinding:
    # kubectl create rolebinding test --role=service-reader --serviceaccount=web:default -n web

    Run a proxy for inter-cluster communications:
    # kubectl proxy

    Try to access the services in the web namespace from anothe terminal:
    # curl localhost:8001/api/v1/namespaces/web/services

    Example 3:
    Create a ClusterRole to access PersistentVolumes:
    ------------------------------------------------

    Create cluster role:
    # kubectl create clusterrole pv-reader --verb=get,list --resource=persistentvolumes

    Create a ClusterRoleBinding for the cluster role:
    # kubectl create clusterrolebinding pv-test --clusterrole=pv-reader --serviceaccount=web:default

    The YAML for a pod that includes a curl and proxy container:
    # cat pod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: curlpod
      namespace: web
    spec:
      containers:
      - image: tutum/curl
        command: ["sleep", "9999999"]
        name: main
      - image: linuxacademycontent/kubectl-proxy
        name: proxy
      restartPolicy: Always

    Create the pod that will allow you to curl directly from the container:
    # kubectl apply -f pod.yaml

    Get the pods in the web namespace:
    # kubectl get pods -n web

    Open a shell to the container:
    # kubectl exec -it curlpod -n web -- sh

    Access PersistentVolumes (cluster-level) from the pod:
    # curl localhost:8001/api/v1/persistentvolumes

    Try deleting the rolebinding and try accessing again from the pod. 





=======================
CNI Plugin. Container Network Interface 
=======================
 - CNI Plugin creates and attaches ENI to the Worker nodes. Also picks up the free IP address on the subnet. This IP is attached to the PODS. 
 - Total NUmber of pods on a node is limitited to the number of ENI that cab ve attached x Number of secondary IP address available on subnet or attachable to the instance.
 - Helps to assign the IP address from the VPC directly to the POD. 
 - CNI plugin is runnign as a pod deployed to all worker nodes as a DaemonSet. DaemonSet ensures a copy of pod is running on all nodes. 
 Two components in the plugin 
 -  CNI plugin : It will wireup the host and pods network stack, 
    L-IPAMD : Local IP address Managed Daemon. Maintans a warm pool of available IP address and Assigning an IP address to a pod. 
     L-IPAMD runs on all nodes as a DaemonSet named aws-node in namespace kube-system. 
       # kubectl get daemonsets --all-namespaces
       # kubectl describe daemonset aws-node -n kube-system
       # kubectl describe pod/aws-node-pwctl -n kube-system
      Max IPs on AWS EKS = min((Max # of ENI per instance x IPv4 addresses per ENI - Number of ENIs(As primary IP is reserved for traffic routing between pods)), "Subnets available free IP").
      L-IPAMD determines the ip address allocation using:
        Max ENI Limit
        Max IP Limit per ENI 
        Primary IP Address 
        Attached ENIs 
        VPC CIDR 
        Used IP Addresses From Kubelete 

  Configuring Network Policies
  ----------------------------
  - Governs how pods communicate each other
  - by default its opened to everyone. 
  - Network policy applies to pods that matches the labelselector or namespaceSelector
  - Ingress, egress can be controlled. 
    
    Download the canal plugin:
    # wget -O canal.yaml https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/canal/canal.yaml

    Apply the canal plugin:
    # kubectl apply -f canal.yaml

    The YAML for a deny-all NetworkPolicy:
    # cat denyall.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: deny-all
    spec:
      podSelector: {}
      policyTypes:
      - Ingress

    Run a deployment to test the NetworkPolicy:
    # kubectl run nginx --image=nginx --replicas=2

    Create a service for the deployment:
    # kubectl expose deployment nginx --port=80

    Attempt to access the service by using a busybox interactive pod:
    # kubectl run busybox --rm -it --image=busybox /bin/sh
      sh# wget --spider --timeout=1 nginx

    The YAML for a pod selector NetworkPolicy:
    # cat podselector.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: db-netpolicy
    spec:
      podSelector:
        matchLabels:
          app: db
      ingress:
      - from:
        - podSelector:
            matchLabels:
              app: web
        ports:
        - port: 5432

    Label a pod to get the NetworkPolicy:
    # kubectl label pods [pod_name] app=db

    The YAML for a namespace NetworkPolicy:
    # cat namespaceSelector.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: ns-netpolicy
    spec:
      podSelector:
        matchLabels:
          app: db
      ingress:
      - from:
        - namespaceSelector:
            matchLabels:
              tenant: web
        ports:
        - port: 5432

    The YAML for an IP block NetworkPolicy:
    # cat ipblock.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: ipblock-netpolicy
    spec:
      podSelector:
        matchLabels:
          app: db
      ingress:
      - from:
        - ipBlock:
            cidr: 192.168.1.0/24

    The YAML for an egress NetworkPolicy:
    # cat egress.yaml 
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: egress-netpol
    spec:
      podSelector:
        matchLabels:
          app: web
      egress:
      - to:
        - podSelector:
            matchLabels:
              app: db
        ports:
        - port: 5432

 =======================
  Creating TLS Certificates
 =======================
  - CA is used to generate a TLS certificte and authenticate with the API Service. 
  - CA bundle is automatically mounted to the pod using the service account under. 
  # ls /var/run/secrets/kubernetes.io/serviceaccount/
    ca.crt	namespace  token
  
  Create and use custom certifate to authenticate to API Server. 
  ------------------------------------------------------------
  1. create CSR. 
Download the binaries for the cfssl tool:
# wget -q --show-progress --https-only --timestamping \
  https://pkg.cfssl.org/R1.2/cfssl_linux-amd64 \
  https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64

Make the binary files executable:
# chmod +x cfssl_linux-amd64 cfssljson_linux-amd64

Move the files into your bin directory:
# sudo mv cfssl_linux-amd64 /usr/local/bin/cfssl
# sudo mv cfssljson_linux-amd64 /usr/local/bin/cfssljson

Check to see if you have cfssl installed correctly:
# cfssl version

Create a CSR file:
# cat <<EOF | cfssl genkey - | cfssljson -bare server
{
  "hosts": [
    "my-svc.my-namespace.svc.cluster.local",
    "my-pod.my-namespace.pod.cluster.local",
    "172.168.0.24",
    "10.0.34.2"
  ],
  "CN": "my-pod.my-namespace.pod.cluster.local",
  "key": {
    "algo": "ecdsa",
    "size": 256
  }
}
EOF

Create a CertificateSigningRequest API object:
# cat <<EOF | kubectl create -f -
apiVersion: certificates.k8s.io/v1beta1
kind: CertificateSigningRequest
metadata:
  name: pod-csr.web
spec:
  groups:
  - system:authenticated
  request: $(cat server.csr | base64 | tr -d '\n')
  usages:
  - digital signature
  - key encipherment
  - server auth
EOF

View the CSRs in the cluster:
# kubectl get csr

View additional details about the CSR:
# kubectl describe csr pod-csr.web

Approve the CSR:
# kubectl certificate approve pod-csr.web

View the certificate within your CSR:
# kubectl get csr pod-csr.web -o yaml

Extract and decode your certificate to use in a file:
# kubectl get csr pod-csr.web -o jsonpath='{.status.certificate}' \
    | base64 --decode > server.crt
 Now this crt file can be used to authenticate with API Server. 

=======================
Secure image in K8 
=======================
 - The image is downloaded from the contianer registry. 
 - Docker stores the credential information in ~/.docker/config.json
 - $ sudo docker images -> shows the current images in the Worker nodes. 
  - Any one having access to the worker node should be able to access the images. This can be locked down by clearing the image after container creation. Image pull policy to always. 

  Change pod to use private Container repo. Example on AWS ECR repo. 
  ---------------------------------------------------------------
  1. Create ECR Repo. 
  2. Login to ECR Repo to generate the docker credentials. 
   # aws ecr get-login --region us-east-1 
   # sudo docker login -u AWS -p XXlongesttoken=  https://12345678.dkr.ecr.us-east-1.amazonaws.com
   # sudo cat .docker/config.json
    - Verify if the credentials are available. 
  
  # k create secret generic regcred  --from-file=.dockerconfigjson=./docker.json --type=kubernetes.io/dockerconfigjson
  # kubectl get secret regcred --output=yaml
   create a secret and verify using above command. 
  
  Create pod pulling image from priv repo using the secrets. 
    # cat ecrpod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: ecr-pod
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: 1234567890.dkr.ecr.us-east-1.amazonaws.com/eks:latest
        command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
        imagePullPolicy: Always
      imagePullSecrets:
      - name: regcred

  Change pod to use private Container repo. Example on Azure repo. 
  ---------------------------------------------------------------
    Location of docker credentials.
    # sudo vim /home/cloud_user/.docker/config.json

    Log in to the Docker Hub:
    # sudo docker login

    View the images currently on your server:
    # sudo docker images

    Pull a new image to use with a Kubernetes pod:
    # sudo docker pull busybox:1.28.4

    Log in to a private registry using the docker login command:
    # sudo docker login -u privreponame -p 'xxxxxxxxxxxxxxx' privreponame.azurecr.io

    View your stored credentials:
    # sudo vim /home/cloud_user/.docker/config.json

    Tag an image in order to push it to a private registry:
    # sudo docker tag busybox:1.28.4 privreponame.azurecr.io/busybox:latest

    Push the image to your private registry:
    # docker push privreponame.azurecr.io/busybox:latest

    Create a new docker-registry secret:
    # kubectl create secret docker-registry acr --docker-server=https://privreponame.azurecr.io --docker-username=privreponame --docker-password='xxxxxxxxxxxxxxx' --docker-email=user@example.com

    Modify the default service account to use your new docker-registry secret:
    # kubectl patch sa default -p '{"imagePullSecrets": [{"name": "acr"}]}'

    The YAML for a pod using an image from a private repository:
    # cat podPrivrepo.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: acr-pod
      labels:
        app: busybox
    spec:
      containers:
        - name: busybox
          image: privreponame.azurecr.io/busybox:latest
          command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
          imagePullPolicy: Always

    Create the pod from the private image:
    # kubectl apply -f podPrivrepo.yaml

    View the running pod:
    # kubectl get pods
 
 Defining Security Contexts
 --------------------------
  - Defining security contexts allows you to lock down your containers, so that only certain processes can do certain things. This ensures the stability of your containers and allows you to give control or take it away.         spec.container.securityContext helps to define the same. 

 # k exec ecr-pod id
uid=0(root) gid=0(root) groups=10(wheel)
 - By default container runs as root. 

 Running a pod with a specific user. 
 # cat pod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: ecr-pod
      labels:
        app: busybox
    spec:
      containers:
      - name: busybox
        image: 1234567890.dkr.ecr.us-east-1.amazonaws.com/eks:latest
        command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
        imagePullPolicy: Always
        securityContext:
          runAsUser: 405
      imagePullSecrets:
      - name: regcred
  # k exec ecr-pod id
  uid=405 gid=0(root)

  - If pods needs to use the kernel feature of nodes, the pod has to run in privileged mode. 
  securityContext.privileged: true 

  Example:
# cat privilegedpod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: privilegedpod
  labels:
    app: busybox
spec:
  containers:
  - name: busybox
    image: 1234567890.dkr.ecr.us-east-1.amazonaws.com/eks:latest
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
    imagePullPolicy: Always
    securityContext:
      privileged: true 
  imagePullSecrets:
  - name: regcred

Check the difference by:
#  k exec normalpod ls /dev
#  k exec privilegedpod ls /dev

$  k exec privilegedpod -- date +%T -s "12:00:00"
12:00:00
$  k exec normalpod -- date +%T -s "12:00:00"
12:00:00
date: can't set date: Operation not permitted
- Defining capabilities will allow pod to perform the action of changing pod time.

# cat capabilityesTime.yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: kernelchange-pod
  spec:
    containers:
    - name: main
      image: alpine
      command: ["/bin/sleep", "999999"]
      securityContext:
        capabilities:
          add:
          - SYS_TIME

  Below example removes the capabilities of chaning ownership:
  # cat dropcapabilities.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: remove-capabilities
    spec:
      containers:
      - name: main
        image: alpine
        command: ["/bin/sleep", "999999"]
        securityContext:
          capabilities:
            drop:
            - CHOWN
  
  Example of a pod container that can’t write to the local filesystem:
    # cat rorootfile.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: readonly-pod
    spec:
      containers:
      - name: main
        image: alpine
        command: ["/bin/sleep", "999999"]
        securityContext:
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: my-volume
          mountPath: /volume
          readOnly: false
      volumes:
      - name: my-volume
        emptyDir:

 Example of a pod that has different group permissions for different containers:
    # cat differentContentxts.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: group-context
    spec:
      securityContext:
        fsGroup: 555
        supplementalGroups: [666, 777]
      containers:
      - name: first
        image: alpine
        command: ["/bin/sleep", "999999"]
        securityContext:
          runAsUser: 1111
        volumeMounts:
        - name: shared-volume
          mountPath: /volume
          readOnly: false
      - name: second
        image: alpine
        command: ["/bin/sleep", "999999"]
        securityContext:
          runAsUser: 2222
        volumeMounts:
        - name: shared-volume
          mountPath: /volume
          readOnly: false
      volumes:
      - name: shared-volume
        emptyDir:
  
  Securing Persistent Key Value Store
  -----------------------------------
  Following example shows how to make use of secret to push TLS key and cert to the pod configuration. 
    Generate a key for your https server:
    # openssl genrsa -out https.key 2048

    Generate a certificate for the https server:
    # openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj /CN=www.example.com

    Create an empty file to create the secret:
    # touch file

    Create a secret from your key, cert, and file:
    # kubectl create secret generic example-https --from-file=https.key --from-file=https.cert --from-file=file

    View the YAML from your new secret:
    # kubectl get secrets example-https -o yaml

    Create the configMap that will mount to your pod:
    # cat configmaphttps.yaml 
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: config
    data:
      my-nginx-config.conf: |
        server {
            listen              80;
            listen              443 ssl;
            server_name         www.example.com;
            ssl_certificate     certs/https.cert;
            ssl_certificate_key certs/https.key;
            ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
            ssl_ciphers         HIGH:!aNULL:!MD5;

            location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;
            }

        }
      sleep-interval: |
        25

    The YAML for a pod using the new secret:
    # cat httpspod.yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: example-https
    spec:
      containers:
      - image: linuxacademycontent/fortune
        name: html-web
        env:
        - name: INTERVAL
          valueFrom:
            configMapKeyRef:
              name: config
              key: sleep-interval
        volumeMounts:
        - name: html
          mountPath: /var/htdocs
      - image: nginx:alpine
        name: web-server
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
          readOnly: true
        - name: config
          mountPath: /etc/nginx/conf.d
          readOnly: true
        - name: certs
          mountPath: /etc/nginx/certs/
          readOnly: true
        ports:
        - containerPort: 80
        - containerPort: 443
      volumes:
      - name: html
        emptyDir: {}
      - name: config
        configMap:
          name: config
          items:
          - key: my-nginx-config.conf
            path: https.conf
      - name: certs
        secret:
          secretName: example-https

    Describe the nginx conf via ConfigMap:
    # kubectl describe configmap

    View the cert mounted on the container:
    # kubectl exec example-https -c web-server -- mount | grep certs

    Use port forwarding on the pod to server traffic from 443:
    # kubectl port-forward example-https 8443:443 &

    Curl the web server to get a response:
    # curl https://localhost:8443 -k



=======================
 Monitoring & Troubleshooting 
=======================
 - Don't have a default monitoring tool, Heapster, Metrics Server are examples.
 - Kubernetes Dashboard. 
 - Logging:  Container logs and Node logs. 
 # kubectl logs podname
 # kubectl logs podname --previous
 Node Logs: 
  Any Container logs StdOut and stderr are logged locally.   
 # journalctl -u kubelet 
 # /var/log/messages, /var/log/kub*, /var/log/container, /var/log/pods 
 Logrotates are used to rotate the logs, Once the logs are rotated, the older logs will not be visible with kubectl logs command.
 
 # kubectl describe node ip-192-168-192-128.ec2.internal

PODS Troubleshooting:
 # aws sts get-caller-identity 
  Check the IAM user in action 
 #  kubectl get pod --output=yaml
 # kubectl describe pod $PODname 
 # kubectl get pod nginx-75f9fb7c5f-lck6j -o go-template="{{range.status.containerStatuses}}{{.lastState.terminated.message}}{{end}}"

 Service Troubleshooting:
  Check if endpoints available 
  # kubectl get endpoints <Service Name>
  Check if the pods matches the labels.  
  # kubectl get pods --selector=app=nginxsvc
  Check containerPort Matches with Service Container port 
   # kubectl describe pods/nginx-75f9fb7c5f-lck6j
   # kubectl describe svc/nginxsvc

Monitoring the Cluster Components
---------------------------------
 - Metrics Server
   Metrics server exposes the metrics via metrics server API. Metrics server discoveres all the nodes in the cluster and quieries each node kubelete for Memory and cpu usage. 

   Install and configure monitoring with Metrics Server: 
   ----------------------------------------------------

    Clone the metrics server repository:
    # git clone https://github.com/linuxacademy/metrics-server

    Install the metrics server in your cluster:
    # kubectl apply -f ~/metrics-server/deploy/1.8+/

    Get a response from the metrics server API:
    # kubectl get --raw /apis/metrics.k8s.io/

    Get the CPU and memory utilization of the nodes in your cluster:
    # kubectl top node

    Get the CPU and memory utilization of the pods in your cluster:
    # kubectl top pods

    Get the CPU and memory of pods in all namespaces:
    # kubectl top pods --all-namespaces

    Get the CPU and memory of pods in only one namespace:
    # kubectl top pods -n kube-system

    Get the CPU and memory of pods with a label selector:
    # kubectl top pod -l run=pod-with-defaults

    Get the CPU and memory of a specific pod:
    # kubectl top pod pod-with-defaults

    Get the CPU and memory of the containers inside the pod:
    # kubectl top pods group-context --containers

  Monitoring the Applications Running within a Cluster
  ----------------------------------------------------
   livenessProbe & readinessProbe can be used to monitor the application. 

   Ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes

    # cat liveness.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: liveness
    spec:
      containers:
      - image: linuxacademycontent/kubeserve
        name: kubeserve
        livenessProbe:
          httpGet:
            path: /
            port: 80

    The YAML for a service and two pods with readiness probes:
    # cat readiness.yaml 
    apiVersion: v1
    kind: Service
    metadata:
      name: nginx
    spec:
      type: LoadBalancer
      ports:
      - port: 80
        targetPort: 80
      selector:
        app: nginx
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginxpd
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:191
        readinessProbe:
          httpGet:
            path: /
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5

    Create the service and two pods with readiness probes:
    # kubectl apply -f readiness.yaml

    Check if the readiness check passed or failed:
    # kubectl get pods

    Check if the failed pod has been added to the list of endpoints:
    # kubectl get ep

    Edit the pod to fix the problem and enter it back into the service:
    # kubectl edit pod nginxpd
      - fix the image to nginx 

    Get the list of endpoints to see that the repaired pod is part of the service again:
    # kubectl get ep

Managing Cluster Component Logs
-------------------------------
  - /var/log/containers is the default directory where containers logs to. 
  - kubelete logs to default syslog location. 
  Example:
    The YAML for a pod that has two different log streams:
    # cat polog.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
    spec:
      containers:
      - name: count
        image: busybox
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}

    Create a pod that has two different log streams to the same directory:
    # kubectl apply -f twolog.yaml

    View the logs in the /var/log directory of the container:
    # kubectl exec counter -- ls /var/log

    The YAML for a sidecar container that will tail the logs for each type:
    # cat sidecarcontainer.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
    spec:
      containers:
      - name: count
        image: busybox
        args:
        - /bin/sh
        - -c
        - >
          i=0;
          while true;
          do
            echo "$i: $(date)" >> /var/log/1.log;
            echo "$(date) INFO $i" >> /var/log/2.log;
            i=$((i+1));
            sleep 1;
          done
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-1
        image: busybox
        args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      - name: count-log-2
        image: busybox
        args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
        volumeMounts:
        - name: varlog
          mountPath: /var/log
      volumes:
      - name: varlog
        emptyDir: {}

    View the first type of logs separately:
    # kubectl logs counter count-log-1

    View the second type of logs separately:
    # kubectl logs counter count-log-2

  Managing Application Logs
  -------------------------
    Get the logs from a pod:
    # kubectl logs nginx

    Get the logs from a specific container on a pod:
    # kubectl logs counter -c count-log-1

    Get the logs from all containers on the pod:
    # kubectl logs counter --all-containers=true

    Get the logs from containers with a certain label:
    # kubectl logs -lapp=nginx

    Get the logs from a previously terminated container within a pod:
    # kubectl logs -p -c nginx nginx

    Stream the logs from a container in a pod:
    # kubectl logs -f -c count-log-1 counter

    Tail the logs to only view a certain number of lines:
    # kubectl logs --tail=20 nginx

    View the logs from a previous time duration:
    # kubectl logs --since=1h nginx

    View the logs from a container within a pod within a deployment:
    # kubectl logs deployment/nginx -c nginx

    Redirect the output of the logs to a file:
    # kubectl logs counter -c count-log-1 > count.log

=======================
Troubleshooting
=======================

Application failures:
--------------------
The YAML for a pod with a termination reason:
# cat podwithreason.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: pod2
spec:
  containers:
  - image: busybox
    name: main
    command:
    - sh
    - -c
    - 'echo "I''ve had enough" > /var/termination-reason ; exit 1'
    terminationMessagePath: /var/termination-reason

One of the first steps in troubleshooting is usually to describe the pod:
# kubectl describe po pod2

The YAML for a liveness probe that checks for pod health:
# cat podwithhealth.yaml 
apiVersion: v1
kind: Pod
metadata:eiifcckikhtdjihtjkdlnnrfiiuvrhufegvjvcvtukkk

  name: liveness
spec:
  containers:
  - image: linuxacademycontent/kubeserve
    name: kubeserve
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8081

View the logs for additional detail:
# kubectl logs pod-with-defaults

Export the YAML of a running pod, in the case that you are unable to edit it directly:
# kubectl get po pod-with-defaults -o yaml --export > defaults-pod.yaml

Edit a pod directly (i.e., changing the image):
# kubectl edit po nginx

  Troubleshooting Cluster failures:
  ---------------------------------
    Check the events in the kube-system namespace for errors:
    # kubectl get events -n kube-system

    Get the logs from the individual pods in your kube-system namespace and check for errors:
    # kubectl logs [kube_scheduler_pod_name] -n kube-system

    Check the status of the Docker service:
    # sudo systemctl status docker

    Start up and enable the Docker service, so it starts upon bootup:
    # sudo systemctl enable docker && systemctl start docker

    Check the status of the kubelet service:
    # sudo systemctl status kubelet

    Start up and enable the kubelet service, so it starts up when the machine is rebooted:
    # sudo systemctl enable kubelet && systemctl start kubelet

 Troubleshooting Worker Node Failure
 -----------------------------------
Listing the status of the nodes should be the first step:
# kubectl get nodes

Find out more information about the nodes with kubectl describe:
# kubectl describe nodes chadcrowell2c.mylabserver.com

You can try to log in to your server via SSH:
# ssh mynode.example.com

Get the IP address of your nodes:
# kubectl get nodes -o wide

Use the IP address to further probe the server:
# ssh cloud_user@<IP address>

Generate a new token after spinning up a new server:
# sudo kubeadm token generate

Create the kubeadm join command for your new worker node:
# sudo kubeadm token create [token_name] --ttl 2h --print-join-command

View the journalctl logs:
# sudo journalctl -u kubelet

View the syslogs:
# sudo more syslog | tail -120 | grep kubelet


 Troubleshooting Networking
 --------------------------
Run a deployment using the container port 9376 and with three replicas:
# kubectl run hostnames --image=k8s.gcr.io/serve_hostname \
                        --labels=app=hostnames \
                        --port=9376 \
                        --replicas=3

List the services in your cluster:
# kubectl get svc

Create a service by exposing a port on the deployment:
# kubectl expose deployment hostnames --port=80 --target-port=9376

Run an interactive busybox pod:
# kubectl run -it --rm --restart=Never busybox --image=busybox:1.28 sh

From the pod, check if DNS is resolving hostnames:
# nslookup hostnames

From the pod, cat out the /etc/resolv.conf file:
# cat /etc/resolv.conf

From the pod, look up the DNS name of the Kubernetes service:
# nslookup kubernetes.default

Get the JSON output of your service:
# kubectl get svc hostnames -o json

View the endpoints for your service:
# kubectl get ep

Communicate with the pod directly (without the service):
# wget -qO- 10.244.1.6:9376

Check if kube-proxy is running on the nodes:
# ps auxw | grep kube-proxy

Check if kube-proxy is writing iptables:
# iptables-save | grep hostnames

View the list of kube-system pods:
# kubectl get pods -n kube-system

Connect to your kube-proxy pod in the kube-system namespace:
# kubectl exec -it kube-proxy-cqptg -n kube-system -- sh
# iptables-save 

Change the CNI Plugin. 
Delete the flannel CNI plugin:
# kubectl delete -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml

Apply the Weave Net CNI plugin:
# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"


=======================
 Resource Quota
=======================

$ kubectl create quota myrq --hard=cpu=400,memory=1G,pods=2 
resourcequota/myrq created

$ k describe quota
Name:       myrq
Namespace:  default
Resource    Used   Hard
--------    ----   ----
cpu         1      400
memory      500Mi  1G
pods        1      2

$ k run nginx --image=nginx --restart='Never' --limits='cpu=1000,memory=500Mi'
Error from server (Forbidden): pods "nginx" is forbidden: exceeded quota: myrq, requested: cpu=1k,memory=500Mi, used: cpu=1,memory=500Mi, limited: cpu=400,memory=1G


=======================
Commands:
=======================
> Check the cluster components status.
--------------------------------------
 # kubectl get cs
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok                   
scheduler            Healthy   ok                   
etcd-0               Healthy   {"health": "true"}  

> Create a cluster from the image:
 # kubectl run my-web --image=nginx --port=80

> Check the deployments
 # kubectl get deployments 
 # kubectl get pods
    NAME                      READY     STATUS    RESTARTS   AGE
    my-web-857d97b7cd-pcr4b   1/1       Running   0          1m

> Create a service, Expose a running pod exposed to external. In this case the nginx pod. 
 # kubectl expose deployment my-web --target-port=80 --type=NodePort
    service "my-web" exposed
> Find details of the service 
 # kubectl get svc
    NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    my-web            NodePort    10.100.64.56     <none>        80:30778/TCP   9s
> Find the port number on which the service is listening.
 # kubectl get svc my-web -o go-template='{{(index .spec.ports 0).nodePort}}'

> Deploying multi tier application:
-----------------------------------
Client -> WebServer -> DBServer 

# cat webserver.yaml
apiVersion: "v1"
kind: Pod   # -> type of entity to create on K8 node.
metadata:
  name: web  # -> Name of the pod 
  labels:     # -> Labels to be used for other services  
    name: web
    app: demo
    env: staging
spec:
  containers:     #-> collection of containers 
    - name: web
      image: janakiramm/web 
      ports:
        - containerPort: 5000
          name: http
          protocol: TCP

# cat redisdb.yaml 
apiVersion: "v1"
kind: Pod
metadata:
  name: redis
  labels:
    name: redis
    app: demo
    env: staging
spec:
  containers:
    - name: redis
      image: redis:latest
      ports:
        - containerPort: 6379         
          protocol: TCP

# cat redisService.yaml 
apiVersion: v1
kind: Service  
metadata:
  name: redis
  labels:
    name: redis
    app: demo  
spec:
  ports:
  - port: 6379   # -> Service port 
    name: redis  # -> 
    targetPort: 6379 # -> Targeting the 6379 of all the pods. 
  selector:        # -> This will make the condition to choose the right pod to reach the traffic on targetPort.
    name: redis
    app: demo

# cat webservice.yaml 
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    name: web
    app: demo
spec:
  selector:
    name: web 
  type: NodePort
  ports:
   - port: 80
     targetPort: 5000
     protocol: TCP

> Create the DataBase pod first. 
-------------------------------
# kubectl create -f redisdb.yaml 
    pod "redis" created

> Create the service, so that it can be discovered by the webservice. (The webservice from the image "image: janakiramm/web " has coded to reach redis on the hostname "redis". The Redis service has the Spec configured to have the targetPort reaching the pod with the name "redis" ) 
# kubectl create -f redissvc.yaml 
    service "redis" created

# kubectl create -f webapp.yaml 
    pod "web" created
# kubectl get pods
    NAME                      READY     STATUS    RESTARTS   AGE
    redis                     1/1       Running   0          5m
    web                       1/1       Running   0          2m

> Login to the webpod and see if everything is fine. 
# kubectl exec -it web /bin/bash
root@web:/usr/src/app# ls
app.py	build.sh  flask  requirements.txt
# ping redis
PING redis.default.svc.cluster.local (10.100.70.198): 56 data bytes
# curl localhost:5000
Hello Container World! I have been seen 1 times. 
 - This makes sure that the application is working locally. That means from inside the Container. 

 Now lets create a service for web application. 

# kubectl get svc
    NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    redis             ClusterIP   10.100.70.198    <none>        6379/TCP       10m
    web               NodePort    10.100.169.184   <none>        80:31360/TCP   8s

# kubectl describe svc web 
    Name:                     web
    Namespace:                default
    Labels:                   app=demo
                            name=web
    Annotations:              <none>
    Selector:                 name=web
    Type:                     NodePort
    IP:                       10.100.169.184  # -> The service should be able access from this IP. 
    Port:                     <unset>  80/TCP
    TargetPort:               5000/TCP
    NodePort:                 <unset>  31360/TCP
    Endpoints:                172.31.60.14:5000
    Session Affinity:         None
    External Traffic Policy:  Cluster
    Events:                   <none>


Replication Controller Object: 
------------------------------
# cat myweb-rc.yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: web
  labels:
    name: web
    app: demo
spec:
  replicas: 2 # -> Desired number of pods to be running.
  template:
    metadata:
      labels:
        name: web
    spec:
      containers:
        - name: web
          image: janakiramm/web
          ports:
            - containerPort: 5000
              name: http
              protocol: TCP

# kubectl create -f myweb-rc.yaml 
replicationcontroller "web" created

# kubectl get rc
NAME           DESIRED   CURRENT   READY     AGE
web            2         2         2         6s

Evenif the pod is deleted, the ReplicationController will recreate the pods. 

# kubectl delete pod web-b69zd 
pod "web-b69zd" deleted

#  kubectl get pods

To scale the pod up or down, can run below command. 
# kubectl scale rc web --replicas=10
replicationcontroller "web" scaled


Create a Single container POD:
------------------------------

# cat mypod.yaml. 
apiVersion: "v1"
kind: Pod # -> what kind of object is creating 
metadata:
  name: mypod
  labels:
    app: demo
    env: test 
spec: # -> where define the specifications of the object. 
  containers:
    - name: nginx
      image: nginx
      ports:
        - name: http
          containerPort: 80
          protocol: TCP
  
# kubectl create -f mypod.yaml 
pod "mypod" created

# kubapp]$ kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
mypod                     1/1       Running   0          6s

# kubectl describe pods mypod 

 Now expose the pod to outside network. 

 # kubectl expose pod mypod --type=NodePort 
 # kubectl describe svc mypod
Name:                     mypod
Namespace:                default
Labels:                   app=demo
                          env=test
Annotations:              <none>
Selector:                 app=demo,env=test
Type:                     NodePort
IP:                       10.100.135.172
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32120/TCP
Endpoints:                172.31.63.239:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>


Create a Multi container POD with a sample app:
----------------------------------------------

> Create the Pods. 
MySQL POD: 
# cat mydb.yaml
apiVersion: "v1"
kind: Pod
metadata:
  name: mysql
  labels:
    name: mysql
    app: demo
spec:
  containers:
    - name: mysql
      image: mysql:latest
      ports:
        - containerPort: 3306         
          protocol: TCP
      env: 
        - 
          name: "MYSQL_ROOT_PASSWORD"
          value: "password"

 WebServer POD: 

 # cat mywebPod.yaml
 apiVersion: "v1"
kind: Pod
metadata:
  name: web1
  labels:
    name: web
    app: demo
spec:
  containers:
    - name: redis
      image: redis
      ports:
        - containerPort: 6379
          name: redis
          protocol: TCP
    - name: python
      image: janakiramm/py-red
      env:       
        - name: "REDIS_HOST"
          value: "localhost"  # -> The Python pod and redis pod share same network name space as the applicaiton should be able to access the redis using locahost. 
      ports:
        - containerPort: 5000
          name: http
          protocol: TCP 

> Create Services: 
# cat musqlService.yamlapiVersion: v1
kind: Service
metadata:
  name: mysql
  labels:
    name: mysql
    app: demo  
spec:
  ports:
  - port: 3306
    name: mysql
    targetPort: 3306
  selector:
    name: mysql
    app: demo

# cat webservice.yaml 
apiVersion: v1
kind: Service
metadata:
  name: web
  labels:
    name: web
    app: demo
spec:
  selector:
    name: web 
  type: NodePort
  ports:
   - port: 80
     targetPort: 5000
     protocol: TCP

Now the configuration is ready and Deploy:

First create DB and applicaiton PODs:
# kubectl create -f mydb.yaml 
pod "mysql" created

# kubectl create -f mywebPod.yaml
pod "web1" created

# kubectl get pods
NAME                      READY     STATUS    RESTARTS   AGE
mysql                     1/1       Running   0          29s



Expose the mysql service:

=======================================
 Kubernetes Security Primitives
=======================================
 - API authentication is following by authentication, authorization and Adminssion Controll the request. 
  - validates wheather the request is coming from a normal user or service account. 
  - Kubernetes doesn't have a normal user Objects and user accounts cannot be added to cluster throught he API calls. 
  - Service accounts are used to manage the identity of request coming from pods to API server. 
  # kubectl get serviceaccount
    - lists all the service accounts 
  # k create sa jenkins
    NAME      SECRETS   AGE
    default   1         4d1h
    jenkins   1         45h
   Creates a service account and secret for the service account.
  
  Secret holds the public certificate authoritry of the API server and a signed json  web token 

  #  kg sa jenkins -o yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      creationTimestamp: "2019-07-25T05:49:25Z"
      name: jenkins
      namespace: default
      resourceVersion: "263634"
      selfLink: /api/v1/namespaces/default/serviceaccounts/jenkins
      uid: f56fd1ee-ae9f-11e9-9839-0a51bfa1d940
    secrets:
    - name: jenkins-token-sgkkc

  $ kg secret jenkins-token-sgkkc
    NAME                  TYPE                                  DATA   AGE
    jenkins-token-sgkkc   kubernetes.io/service-account-token   3      45h

 $ kd secret/jenkins-token-sgkkc
    Name:         jenkins-token-sgkkc
    Namespace:    default
    Labels:       <none>
    Annotations:  kubernetes.io/service-account.name: jenkins
                  kubernetes.io/service-account.uid: f56fd1ee-ae9f-11e9-9839-0a51bfa1d940

    Type:  kubernetes.io/service-account-token

    Data
    ====
    ca.crt:     1025 bytes
    namespace:  7 bytes
    token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImplbmtpbnMtdG9rZW4tc2dra2MiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiamVua2lucyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImY1NmZkMWVlLWFlOWYtMTFlOS05ODM5LTBhNTFiZmExZDk0MCIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmplbmtpbnMifQ.mGp1rnHe5Xv41L8Hg0gwS-o5cvYCNRto2J4drFhCynSnAP31kl3bg8ANHMZ8288M3Tn1oasPhKrPi8YrFJxJKOc_Uj_asQx8jTLMczyaXLtC6nKCaPindpaWm0W-TyL2qbDX8h51iKomJ4lJPHHo-ym_3ojMkmQI1D9XFisz0ZnGNnLounx9kV9zzZunTv1WXJ28i-y_3Lttqx865jY79SdcjRDygn2Sn_rwtjyp1NNIi-rDXokphpPIYUZXwGf58ll5P6o3cYFGTZNxWcYwIVmxQPUE-QInM90u0cZTVphLswDNXNnnAAmFGmFdzkC7vbfIGjhsSET7rYcCm03qdQ

  By default when a pod is added, it uses the service account default. 
  # kg pod mypod -o yaml | grep -i service
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
    enableServiceLinks: true
    serviceAccount: default
    serviceAccountName: default

    Assigning service account to a Pod. 
    ----------------------------------
  # cat sasamplepod.yaml 
    apiVersion: v1
    kind: Pod
    metadata:
      name: busybox
      namespace: default
    spec:
      serviceAccountName: jenkins
      containers:
      - image: busybox:1.28.4
        command:
          - sleep
          - "3600"
        imagePullPolicy: IfNotPresent
        name: busybox
      restartPolicy: Always
  #  kg pod busybox -o yaml | grep -i serviceacc
        {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"busybox","namespace":"default"},"spec":{"containers":[{"command":["sleep","3600"],"image":"busybox:1.28.4","imagePullPolicy":"IfNotPresent","name":"busybox"}],"restartPolicy":"Always","serviceAccountName":"jenkins"}}
      - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
    serviceAccount: jenkins
    serviceAccountName: jenkins

    - Now that you can access the application running on this pod, add kubernetes cli plugin, enter the token and this application can control the pods using the service account token. 

Additional users to authenticate with Kubernetes cluster. 
------------------------------------------------------
$ k config view  
 Shows the current configuration of the kubectl api tool. 

Configure additional user to access Kubernetes. 
----------------------------------------------

  On Kubernetes= Master/first configured Client:
    # kubectl config view

    Set new credentials for your cluster:
    # kubectl config set-credentials chad --username=chad --password=password

    Create a role binding for anonymous users (not recommended):
    # kubectl create clusterrolebinding cluster-system-anonymous --clusterrole=cluster-admin --user=system:anonymous

    SCP the certificate authority to your workstation or server:
    # scp /etc/kubernetes/pki/ca.crt user@[pub-ip-of-remote-server]:~/


  On new Client or remote system set cluster address and authentication:
    # kubectl config set-cluster kubernetes --server=https://172.31.41.61:6443 --certificate-authority=ca.crt --embed-certs=true

    Set the credentials for Chad:
    # kubectl config set-credentials chad --username=chad --password=password

    Set the context for the cluster:
    # kubectl config set-context kubernetes --cluster=kubernetes --user=chad --namespace=default

    Use the context:
    # kubectl config use-context kubernetes

    Run the same commands with kubectl:
    # kubectl get nodes




  Secrets in Kubernetes
  ----------------------

Service Accounts:
 - A service account provides an identity for processes that run in a Pod.

# kubectl get pods/guestbook-9k9bb -o yaml
spec:
  serviceAccount: default
  serviceAccountName: default


Secret file from Kubernetes nodes are mounted to all pods, from location:
 -> /var/lib/kubelet/pods/<7382afe2-0276-11e9-8174-1205d43758d2>/

#  docker inspect <pod conainer ID> | grep mount 
  "Mounts": [
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/d2b6e799-0276-11e9-8174-1205d43758d2/volumes/kubernetes.io~secret/default-token-wkfmn",
                "Destination": "/var/run/secrets/kubernetes.io/serviceaccount",
                "Mode": "ro",
                "RW": false,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/d2b6e799-0276-11e9-8174-1205d43758d2/etc-hosts",
                "Destination": "/etc/hosts",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            },
            {
                "Type": "bind",
                "Source": "/var/lib/kubelet/pods/d2b6e799-0276-11e9-8174-1205d43758d2/containers/guestbook/01141970",
                "Destination": "/dev/termination-log",
                "Mode": "",
                "RW": true,
                "Propagation": "rprivate"
            }
        ],

The default service account shows the service account that pod is used 
# kubectl describe secret
Name:         default-token-wkfmn
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 40ebb1f1-0274-11e9-8174-1205d43758d2

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4td2tmbW4iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjQwZWJiMWYxLTAyNzQtMTFlOS04MTc0LTEyMDVkNDM3NThkMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.hJ2IGW-RGEt87NUyvf8_Co11LlOvx-eeQt8oDPr9Qs9P2H6cGxjsuCHicErQApxCjBInUeeJ9EuTJOWu1OrtCuCFKMrv3SvalK36zv1fv0WPRMHv8yLhajUFDcQntw6t1Ta5JVGn_LNu8hANl1xiwBDICU8ZGsfvK-wS-MdLws4B_bsM7HTF38Nz33DVZgCUQyCVUV0dmHeD-WwOInmh5ExTSliyL_F6MfsHIrSeDUeWXvKDTjh8YUD4wxNx8hkeEY0ZtrN_xnvoThjChchCtK7XrImRX5dDNxO0s9bBepOmI2mwkvq7vpkZIvQGtKM3F-UwmlonqfgkHUMGQUnteg


# Describe all service accounts: 
 > "kubectl describe secret --all-namespaces"



================================================
ALB INGRESS CONTROLLER 
================================================
# wget https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.0/docs/examples/rbac-role.yaml
  kubectl apply -f rbac-role.yaml 

 # curl -sS "https://raw.githubusercontent.com/kubernetes-sigs/aws-alb-ingress-controller/v1.0.0/docs/examples/alb-ingress-controller.yaml" > alb-ingress-controller.yaml
 #  vim alb-ingress-controller.yaml 
 #  kubectl apply -f alb-ingress-controller.yaml 
 Get logs of ingress controller.
# kubectl logs -n kube-system $(kubectl get po -n kube-system | egrep -o alb-ingress[a-zA-Z0-9-]+)
 # kubectl get ing

=======================================
 EDIT core DNS  
=======================================

# kubectl -n kube-system get cm coredns -o yaml

# kubectl -n kube-system edit cm coredns -o yaml
apiVersion: v1
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local {
          pods insecure
          upstream
          fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        proxy . /etc/resolv.conf
        cache 30
    }
    MyDomain.local:53 {
        errors
        cache 30
        proxy . 10.150.0.1 10.151.0.2 
    }

     This will make sure the request to domain is always forwarded to the name servers. 
By default, the pods sends request to Cluster DNS IP (Which is Core DNS, then not availble it proxies to the DNS name at nodes.)

Restart the Pods 
# kubectl get pods -n kube-system -l k8s-app=kube-dns
# kubectl -n kube-system delete pod -l k8s-app=kube-dns


# Run an cURL image to test pod connectivity:
kubectl run  curl --image=tutum/curl --replicas=1 sleep 1d



=======================================
RBAC : https://www.youtube.com/watch?v=CnHTCTP8d48&feature=youtu.be
aws-auth Config-Map 
=======================================
 mapUsers:
 - userARN: arn:aws:iam::101010101010:user/Alice
   username: Alice
   groups:
   - system:masters 

=======================================
 Node Taints/Tolerations 
 Pod Affinities/Anti-Affinities 



=======================================
 # COMMAND Reference Links / POD 
 =======================================
 Run busy box and get to shell.
 # kubectl run -i --tty busybox --image=busybox --restart=Never -- sh  

List all resources supported by K8 cluster. 
 #  kubectl api-resources -o wide


 =================
 Custom Metrics 
 =================

 Installing Prometheus: 
 ---------------------

 Install Helm to setup Prometheus:
 ---------------------------------
 # cd ~/environment
 # curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
 # chmod +x get_helm.sh

 # cat helmrbac.yaml 
      ---
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: tiller
        namespace: kube-system
      ---
      apiVersion: rbac.authorization.k8s.io/v1beta1
      kind: ClusterRoleBinding
      metadata:
        name: tiller
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: cluster-admin
      subjects:
        - kind: ServiceAccount
          name: tiller
          namespace: kube-system
  # helm init --service-account tiller
  # kubectl create namespace prometheus
  # kubectl get all -n prometheus
  # kubectl get --raw /metrics
  # helm install stable/prometheus --name prometheus --namespace prometheus --set alertmanager.persistentVolume.storageClass="gp2",server.persistentVolume.storageClass="gp2"
  # kubectl get pods -n prometheus


 Enable custom metrics using Prometheus Adaptor:
 ----------------------------------------------
 Ref: https://itnext.io/horizontal-pod-autoscale-with-custom-metrics-8cb13e9d475
   # git clone https://github.com/helm/charts.git
   # cd charts/
   # helm install --name prometheuscustmetric --namespace prometheus stable/prometheus-adapter
   # kubectl get all -n prometheus
   # kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
   # kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1 | jq .
    {
    "kind": "APIResourceList",
    "apiVersion": "v1",
    "groupVersion": "custom.metrics.k8s.io/v1beta1",
    "resources": []
    }
    https://github.com/DirectXMan12/k8s-prometheus-adapter
 Ref: https://github.com/luxas/kubeadm-workshop#deploying-the-prometheus-operator-for-monitoring-services-in-the-cluster


Assume role with IAM 
# aws --region us-east-1 eks update-kubeconfig --name ekscluster-Test --role-arn arn:aws:iam::12345678:role/Devops-sampleRole

=================
 EXAM Prep 
=================
 - 180 mins long
 - kubectl config use-context k8s 
  Links can be opened. 
    - https://kubernetes.io/docs/home/ 
    - https://github.com/kubernets
    - https://kubernetes.io/blog 
 - web cam, micro phone, ID card. 

  - Use kubectl completion
  # source <(kubectl completion bash)
  # echo "source <(kubectl completion bash)" >> ~/.bashrc
  https://kubernetes.io/docs/reference/kubectl/cheatsheet/#kubectl-autocomplete

  - Use kubectl help
  # kubectl help
  https://kubernetes.io/docs/reference/kubectl/overview/#syntax

  - Use kubectl explain
  Get documentation of various resources. For instance pods, nodes, services, etc.
  # kubectl explain deployments
  https://kubernetes.io/docs/reference/kubectl/overview/#operations

  - Switch Between Multiple Contexts
  # kubectl config use-context
  https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/#define-clusters-users-and-contexts



=================
EXAMPLES 
=================

## DEPLOYMENT WITH READINESS AND LIVENESS PROBE 
-----------------------------------------------
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: null
  generation: 1
  labels:
    run: webapp
  name: webapp
  namespace: web
  selfLink: /apis/extensions/v1beta1/namespaces/web/deployments/webapp
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      run: webapp
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        run: webapp
    spec:
      containers:
      - image: nginx
        imagePullPolicy: Always
        name: webapp
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8081
        readinessProbe:
           httpGet:
             path: /
             port: 80
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status: {}

## POD WITH PERSISTENTVOLUME CLAIM
-----------------------------------
# pod yaml 
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: data-pod
  name: data-pod
  namespace: web
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - sleep 3600
    image: busybox
    name: data-pod
    volumeMounts: 
    - name: temp-data
      mountPath: /tmp/data
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
  volumes:
  - name: temp-data
    persistentVolumeClaim:
      claimName: data-pvc
status: {}
# pvc yaml
cat pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: data-pvc
  namespace: web
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 256Mi
  storageClassName: local-storage

# kg pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS    REASON   AGE
data-pv   1Gi        RWO            Retain           Available           local-storage            81m

