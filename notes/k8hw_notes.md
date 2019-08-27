# KUBERNETES THE HARDWAY
 Ref: 
 https://github.com/kelseyhightower/kubernetes-the-hard-way
 
 ## Build your own Kubernetes Cluster: 

 ## What will the Kubernetes Cluster Architecture Look Like?
  ----------------------------------------------------------
In order to fully understand the purpose behind many of the steps involved in Kubernetes the Hard Way, it is important to be aware of the architecture that you will implement. This lesson introduces you to the end-state architecture of the Kubernetes cluster which you will be building as you proceed with this course. It will provide you with the context that you need in order to understand some of the later steps.

 Refer the diagram: 
  > The arch should look like by the end of configuration. 
   2 workers, 1 Loadbalancer and 2 controllers 

   ### Componenets 
    * Controller will have:
      > etcd
      > kube-apiserver
      > kube-controller-manager
      > kube-scheduler 
      - Additionally a Loadbalancer between kube-apiserver 

    * Worker node will have:
      > kubelet 
      > kube-proxy 
      > containerd 

    Client Tools:
    ------------
    > cfssl to setup the ssl certificate 
    > kubectl 
    List the installation guide here: https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/02-client-tools.md


 ## Why Do We Need a CA and TLS Certificates?
 ------------------------------------------
  Provisioning a CA Certificate and Generating a TLS certificate. 

  Why need CA and TLS Certificates: 
   - CA and TLS relation -> Provision the CA and use the CA to generate several TLS Certificate.
     - Certificates are used to confirm the identity. They are used to provde that you are who you say you are. Kind of more secured way of password to identify and authenticate. 
     - A CA provides the ability to confirm that a certificate is valid. A CA can be used to validate any certifate that was issued using that CA.
   - Kubernetes uses certificates for variety of funtion. The common certificates are 
      - Client Certificates 
        These certificates provide cluent authentication for various users: admin, kube-controller-manager, kube-proxy, kube-scheduler and the kubelet client on each worker node. This is sort of a password, like the components are used to authenticate with kube api server. 
      - Kubernetes API Server Certificate 
        This is the TLS certificate for the Kubernertes API. When client is talking to the API, the TLS is used to confirm the identity of Kubernetes API server. 
      - Service Account Key Pair 
        Kubernertes uses a ceritificate to sign service account tokens, so we need to provide a ceritificate for that purpose. 

 ## Generate TLS Certificate for building K8 cluster. 

   ### Step 1. Provisioning the Certificate Authority. 
  --------------------------------------------------
In order to generate the certificates needed by Kubernetes, you must first provision a certificate authority. This lesson will guide you through the process of provisioning a new certificate authority for your Kubernetes cluster. After completing this lesson, you should have a certificate authority, which consists of two files: ca-key.pem and ca.pem.

Here are the commands used in the demo:

```
cd ~/
mkdir kthw
cd kthw/
```
UPDATE: cfssljson and cfssl will need to be installed. To install, complete the following commands:
```
sudo curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
sudo curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
sudo curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
sudo chmod +x /bin/cfssl*
```
Use this command to generate the certificate authority. Include the opening and closing curly braces to run this entire block as a single command.
```
{

cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json << EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "US",
      "L": "Portland",
      "O": "Kubernetes",
      "OU": "CA",
      "ST": "Oregon"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```
ca-key.pem is the private key 
ca.pem is the public CA that will be moved to each and every components to validate the TLS certificates. 

  ### Step 2. Generating Client Certificates. 
  -------------------------------------------
      Several Client cetiticates needed to be generated. Now that you have provisioned a certificate authority for the Kubernetes cluster, you are ready to begin generating certificates. The first set of certificates you will need to generate consists of the client certificates used by various Kubernetes components. In this lesson, we will generate the following client certificates: admin, kubelet (one for each worker node), kube-controller-manager, kube-proxy, and kube-scheduler. After completing this lesson, you will have the client certificate files which you will need later to set up the cluster.

      Here are the commands used in the demo. The command blocks surrounded by curly braces can be entered as a single command:
    ```
    cd ~/kthw
    ```
      #### Admin Client certificate:
      -------------------------
    ```
    {

    cat > admin-csr.json << EOF
    {
      "CN": "admin",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:masters",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      admin-csr.json | cfssljson -bare admin

    }
    ```
    #### Kubelet Client certificates:
    --------------------------------
    These are just a little more conmplicated. Because these certifate is signed for specific host (Node). Kubelet Client certificates. Be sure to enter your actual cloud server values for all four of the variables at the top:
    ```
    WORKER0_HOST=<Public hostname of your first worker node cloud server>
    WORKER0_IP=<Private IP of your first worker node cloud server>
    WORKER1_HOST=<Public hostname of your second worker node cloud server>
    WORKER1_IP=<Private IP of your second worker node cloud server>

    {
    cat > ${WORKER0_HOST}-csr.json << EOF
    {
      "CN": "system:node:${WORKER0_HOST}",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:nodes",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=${WORKER0_IP},${WORKER0_HOST} \
      -profile=kubernetes \
      ${WORKER0_HOST}-csr.json | cfssljson -bare ${WORKER0_HOST}

    cat > ${WORKER1_HOST}-csr.json << EOF
    {
      "CN": "system:node:${WORKER1_HOST}",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:nodes",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=${WORKER1_IP},${WORKER1_HOST} \
      -profile=kubernetes \
      ${WORKER1_HOST}-csr.json | cfssljson -bare ${WORKER1_HOST}

    }
    ```
    #### Controller Manager Client certificate:
    ------------------------------------------
    ```
    {
    cat > kube-controller-manager-csr.json << EOF
    {
      "CN": "system:kube-controller-manager",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:kube-controller-manager",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

    }
    ```
    #### Kube Proxy Client certificate:
    ----------------------------------
    ```
    {
    cat > kube-proxy-csr.json << EOF
    {
      "CN": "system:kube-proxy",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:node-proxier",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      kube-proxy-csr.json | cfssljson -bare kube-proxy

    }
    ```
    #### Kube Scheduler Client Certificate:
    --------------------------------------
    ```
    {

    cat > kube-scheduler-csr.json << EOF
    {
      "CN": "system:kube-scheduler",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "system:kube-scheduler",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      kube-scheduler-csr.json | cfssljson -bare kube-scheduler

    }
    ```

  ### Step 3. Generating the Kubernetes API Server Certificate. 
  -------------------------------------------------------------
    We have generated all of the the client certificates our Kubernetes clister will need, but we also need a server certificate for the Kubernetes API. In this lesson, we will generate one, signed with all of the hostnames and IPs that may be used later in order to access the Kubernetes API. After completing this lesson, you will have a Kubernetes API server certificate in the form of two files called kubernetes-key.pem and kubernetes.pem.

    Here are the commands used in the demo. Be sure to replace all the placeholder values in CERT_HOSTNAME with their real values from your cloud servers:
    ```
    cd ~/kthw
    CERT_HOSTNAME=10.32.0.1,<controller node 1 Private IP>,<controller node 1 hostname>,<controller node 2 Private IP>,<controller node 2 hostname>,<API load balancer Private IP>,<API load balancer hostname>,127.0.0.1,localhost,kubernetes.default
    >  example:
    >  CERT_HOSTNAME=10.96.0.1,192.168.192.150,ip-192-168-192-150.ec2.internal,127.0.0.1,localhost,kubernetes.default,kubernetes.default.cluster.local,cluster.local

    {

    cat > kubernetes-csr.json << EOF
    {
      "CN": "kubernetes",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "Kubernetes",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -hostname=${CERT_HOSTNAME} \
      -profile=kubernetes \
      kubernetes-csr.json | cfssljson -bare kubernetes

    }
    ```
  ### Step 4. Generating the Service Account Key Pair
  ---------------------------------------------------
    Kubernetes provides the ability for service accounts to authenticate using tokens. It uses a key-pair to provide signatures for those tokens. In this lesson, we will generate a certificate that will be used as that key-pair. After completing this lesson, you will have a certificate ready to be used as a service account key-pair in the form of two files: service-account-key.pem and service-account.pem. Used to authenticate with the service account created in the Kubernertes. 

    Here are the commands used in the demo:
    ```
    cd ~/kthw

    {

    cat > service-account-csr.json << EOF
    {
      "CN": "service-accounts",
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "US",
          "L": "Portland",
          "O": "Kubernetes",
          "OU": "Kubernetes The Hard Way",
          "ST": "Oregon"
        }
      ]
    }
    EOF

    cfssl gencert \
      -ca=ca.pem \
      -ca-key=ca-key.pem \
      -config=ca-config.json \
      -profile=kubernetes \
      service-account-csr.json | cfssljson -bare service-account

    }
    ```
  ### Step 5. Distributing the Certificate Files
  ---------------------------------------------
    Now that all of the necessary certificates have been generated, we need to move the files onto the appropriate servers. In this lesson, we will copy the necessary certificate files to each of our cloud servers. After completing this lesson, your controller and worker nodes should each have the certificate files which they need.

    Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.

    Move certificate files to the worker nodes:
    ```
    scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem user@<worker 1 public IP>:~/
    scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem user@<worker 2 public IP>:~/
    ```
    Move certificate files to the controller nodes:
    ```
    scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
        service-account-key.pem service-account.pem user@<controller 1 public IP>:~/
    scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
        service-account-key.pem service-account.pem user@<controller 2 public IP>:~/
    ```
 ## Generating Kubeconfig 
  The next step in building a Kubernetes cluster the hard way is to generate kubeconfigs which will be used by the various services that will make up the cluster. In this lesson, we will generate these kubeconfigs. After completing this lesson, you should have a set of kubeconfigs which you will need later in order to configure the Kubernetes cluster.

  Here are the commands used in the demo. Be sure to replace the placeholders with actual values from your cloud servers.

  Create an environment variable to store the address of the Kubernetes API, and set it to the private IP of your load balancer cloud server:

  KUBERNETES_ADDRESS=<load balancer private ip>

  #### Step 1.  Generate a kubelet kubeconfig for each worker node:
  ---------------------------------------------------------------
      ```
      for instance in <worker 1 hostname> <worker 2 hostname>; do
        kubectl config set-cluster kubernetes-the-hard-way \
          --certificate-authority=ca.pem \
          --embed-certs=true \
          --server=https://${KUBERNETES_ADDRESS}:6443 \
          --kubeconfig=${instance}.kubeconfig

        kubectl config set-credentials system:node:${instance} \
          --client-certificate=${instance}.pem \
          --client-key=${instance}-key.pem \
          --embed-certs=true \
          --kubeconfig=${instance}.kubeconfig

        kubectl config set-context default \
          --cluster=kubernetes-the-hard-way \
          --user=system:node:${instance} \
          --kubeconfig=${instance}.kubeconfig

        kubectl config use-context default --kubeconfig=${instance}.kubeconfig
      done
      ```
  #### Step 2. Generate a kube-proxy kubeconfig:
  --------------------------------------------
    ```
    {
      kubectl config set-cluster kubernetes-the-hard-way \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=https://${KUBERNETES_ADDRESS}:6443 \
        --kubeconfig=kube-proxy.kubeconfig

      kubectl config set-credentials system:kube-proxy \
        --client-certificate=kube-proxy.pem \
        --client-key=kube-proxy-key.pem \
        --embed-certs=true \
        --kubeconfig=kube-proxy.kubeconfig

      kubectl config set-context default \
        --cluster=kubernetes-the-hard-way \
        --user=system:kube-proxy \
        --kubeconfig=kube-proxy.kubeconfig

      kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
    }
    ```
  #### Step 3. Generate a kube-controller-manager kubeconfig:
  ----------------------------------------------------------
    ```
    {
      kubectl config set-cluster kubernetes-the-hard-way \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=https://127.0.0.1:6443 \
        --kubeconfig=kube-controller-manager.kubeconfig

      kubectl config set-credentials system:kube-controller-manager \
        --client-certificate=kube-controller-manager.pem \
        --client-key=kube-controller-manager-key.pem \
        --embed-certs=true \
        --kubeconfig=kube-controller-manager.kubeconfig

      kubectl config set-context default \
        --cluster=kubernetes-the-hard-way \
        --user=system:kube-controller-manager \
        --kubeconfig=kube-controller-manager.kubeconfig

      kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
    }
    ```
  #### Step 4. Generate a kube-scheduler kubeconfig:
  ------------------------------------------------
    ```
    {
      kubectl config set-cluster kubernetes-the-hard-way \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=https://127.0.0.1:6443 \
        --kubeconfig=kube-scheduler.kubeconfig

      kubectl config set-credentials system:kube-scheduler \
        --client-certificate=kube-scheduler.pem \
        --client-key=kube-scheduler-key.pem \
        --embed-certs=true \
        --kubeconfig=kube-scheduler.kubeconfig

      kubectl config set-context default \
        --cluster=kubernetes-the-hard-way \
        --user=system:kube-scheduler \
        --kubeconfig=kube-scheduler.kubeconfig

      kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
    }
    ```
  #### Step 5. Generate an admin kubeconfig:
   ---------------------------------------
    ```
    {
      kubectl config set-cluster kubernetes-the-hard-way \
        --certificate-authority=ca.pem \
        --embed-certs=true \
        --server=https://127.0.0.1:6443 \
        --kubeconfig=admin.kubeconfig

      kubectl config set-credentials admin \
        --client-certificate=admin.pem \
        --client-key=admin-key.pem \
        --embed-certs=true \
        --kubeconfig=admin.kubeconfig

      kubectl config set-context default \
        --cluster=kubernetes-the-hard-way \
        --user=admin \
        --kubeconfig=admin.kubeconfig

      kubectl config use-context default --kubeconfig=admin.kubeconfig
    }
    ```
  - This creates the necessary configurations for the system pods and nodes. This will help to setup the kubernetes cluster. 


  > Now that we have generated the kubeconfig files that we will need in order to configure our Kubernetes cluster, we need to make sure that each cloud server has a copy of the kubeconfig files that it will need. In this lesson, we will distribute the kubeconfig files to each of the worker and controller nodes so that they will be in place for future lessons. After completing this lesson, each of your worker and controller nodes should have a copy of the kubeconfig files it needs.

  Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.
  #### Step 6. Move kubeconfig files to the worker nodes:
  ----------------------------------------------
    ```
    scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
    scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/
    ```
  #### Step 7. Move kubeconfig files to the controller nodes:
  ---------------------------------------------------
    ```
    scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
    scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/
    ```
 ## Generating the Data Encryption Config and Key
  
    > Kubernetes allows to encrypts the secrets at rest. Meaning it encrypts the data when its in rest. 
    > This will generate the encryption key and put in the config file, and this will be the kubernetes data encryption below: 
    Ref: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
    One important security practice is to ensure that sensitive data is never stored in plain text. Kubernetes offers the ability to encrypt sensitive data when it is stored. However, in order to use this feature it is necessary to provide Kubernetes with a data encrpytion config containing an encryption key. This lesson briefly discusses what the data encrpytion config is and why it is needed. This will provide you with some background knowledge as you proceed to create a data encrpytion config for your cluster. After completing this lesson, you will have a basic understanding of what a data encrpytion config is and what it is used for.

  ### Step 1. Generate the Kubernetes Data encrpytion config file containing the encrpytion key:
    ```
    ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

    cat > encryption-config.yaml << EOF
    kind: EncryptionConfig
    apiVersion: v1
    resources:
      - resources:
          - secrets
        providers:
          - aescbc:
              keys:
                - name: key1
                  secret: ${ENCRYPTION_KEY}
          - identity: {}
    EOF
    ``` 
    Copy the file to both controller servers:
    ```
    scp encryption-config.yaml user@<controller 1 public ip>:~/
    scp encryption-config.yaml user@<controller 2 public ip>:~/
    ```
 ## Initiate/Install components. 
 ## What is etcd 
    In this section, we will be setting up an etcd cluster, which is a necessary component of our Kubernetes cluster. 
    > ETCD is a distributed key value store to store the data reliably. Makes sure that the data is syncronized between multiple nodes and the data is synced and reliable. 

    In order to fully understand this process, however, it is a good idea to have some idea of what etcd and the role it plays in Kubernetes. This lesson introduces you to etcd and discusses how Kubernetes uses it. After completing this lesson, you will have some background knowledge to prepare you for the task of installing and configuring etcd.

    You can find more information about etcd in the following locations:
        https://coreos.com/etcd/
        https://github.com/coreos/etcd (this GitHub repository also containes the etcd source code)

    Check out the Kubernetes documentation for more information on managing etcd in the context of a Kubernetes cluster:
    Ref:
    https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

    Kubernetes uses etcd to store all the internal data of cluster status. Example, the pods objects. All kubernetes controller will have access to this data via kube api server. This data is reliably synchronized between the etcd instances. 

  ### Creating the etcd Cluster
    Before you can stand up controllers for a Kubernetes cluster, you must first build an etcd cluster across your Kubernetes control nodes. This lesson provides a demonstration of how to set up an etcd cluster in preparation for bootstrapping Kubernetes. After completing this lesson, you should have a working etcd cluster that consists of your Kubernetes control nodes.

  ### Step 1. Install and start etcd 
    Here are the commands used in the demo (note that these have to be run on both controller servers, with a few differences between them):
    ```
    wget -q --show-progress --https-only --timestamping \
      "https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz"
    tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
    sudo mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/
    sudo mkdir -p /etc/etcd /var/lib/etcd
    sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
    ```
    Set up the following environment variables. Be sure you replace all of the <placeholder values> with their corresponding real values:

    ```
    ETCD_NAME=<cloud server hostname>
    (this is going to be a uniqueue name for both the controllers, example, this can be the hostname)
    INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
    INITIAL_CLUSTER=<controller 1 hostname>=https://<controller 1 private ip>:2380,<controller 2 hostname>=https://<controller 2 private ip>:2380
    (this is the list of all the controller IP addresses example: INITIAL_CLUSTER=kiranjith1c.mylabserver.com=https://172.31.6.157:2380,kiranjith2c.mylabserver.com=https://172.31.2.178:2380)
    ```
    Create the systemd unit file for etcd using this command. Note that this command uses the environment variables that were set earlier:
    ```
    cat << EOF | sudo tee /etc/systemd/system/etcd.service
    [Unit]
    Description=etcd
    Documentation=https://github.com/coreos

    [Service]
    ExecStart=/usr/local/bin/etcd \\
      --name ${ETCD_NAME} \\
      --cert-file=/etc/etcd/kubernetes.pem \\
      --key-file=/etc/etcd/kubernetes-key.pem \\
      --peer-cert-file=/etc/etcd/kubernetes.pem \\
      --peer-key-file=/etc/etcd/kubernetes-key.pem \\
      --trusted-ca-file=/etc/etcd/ca.pem \\
      --peer-trusted-ca-file=/etc/etcd/ca.pem \\
      --peer-client-cert-auth \\
      --client-cert-auth \\
      --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
      --listen-peer-urls https://${INTERNAL_IP}:2380 \\
      --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
      --advertise-client-urls https://${INTERNAL_IP}:2379 \\
      --initial-cluster-token etcd-cluster-0 \\
      --initial-cluster ${INITIAL_CLUSTER} \\
      --initial-cluster-state new \\
      --data-dir=/var/lib/etcd
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    Start and enable the etcd service:
    ```
    sudo systemctl daemon-reload
    sudo systemctl enable etcd
    sudo systemctl start etcd
    ```
    You can verify that the etcd service started up successfully like so:
    ```
    sudo systemctl status etcd
    ```
    Use this command to verify that etcd is working correctly. The output should list your two etcd nodes:
    ```
    sudo ETCDCTL_API=3 etcdctl member list \
      --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/ca.pem \
      --cert=/etc/etcd/kubernetes.pem \
      --key=/etc/etcd/kubernetes-key.pem
    ```
 ## Bootstrapping the Kubernetes Control Plane
  This is the set of service that controls k8 services. This is the orchestration and control part of K8 services. 

    Coordinating the scheduling process, fulfilling the replicas of Deployments and Replicasets.

    In this section, we will be setting up a distributed Kubernetes control plane. This lesson introduces the control plane and provides a brief overview of the various components that you will be working with. After completing this lesson, you should have an idea of what the Kubernetes control plane does, and you will have a basic understanding of the components that you will be installing and configuring throughout this section of the course.
    You can find more information on the Kubernetes control plane in the official docs: https://kubernetes.io/docs/concepts/overview/components/#master-components
   
    > Components are:
      * kube-api server 
      * etcd 
      * kube scheduler 
      * kube controller managers
      * Cloud Controller manager (If pods are running on a Cloud Provider) 

  ### Control Plane Architecture Overview
    This lesson provides a brief overview of the architectural end-state of this section of the course. After completing this lesson, you should have an understanding of what this section is seeking to accomplish, and what your Kubernetes cluster will look like after this section is completed. You will then be ready to begin the process of actually building out your own Kubernetes control plane!
    Control plane is going to be installed on two nodes. 
  
  ### Installing Kubernetes Control Plane Binaries
    The first step in bootstrapping a new Kubernetes control plane is to install the necessary binaries on the controller servers. We will walk through the process of downloading and installing the binaries on both Kubernetes controllers. This will prepare your environment for the lessons that follow, in which we will configure these binaries to run as systemd services.
    You can install the control plane binaries on each control node like this:
    ```
        sudo mkdir -p /etc/kubernetes/config
        wget -q --show-progress --https-only --timestamping \
          "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-apiserver" \
          "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-controller-manager" \
          "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-scheduler" \
          "https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl"
        chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
        sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
    ```

  ### Setting up the Kubernetes API Server
    The Kubernetes API server provides the primary interface for the Kubernetes control plane and the cluster as a whole. When you interact with Kubernetes, you are nearly always doing it through the Kubernetes API server. This lesson will guide you through the process of configuring the kube-apiserver service on your two Kubernetes control nodes. After completing this lesson, you should have a systemd unit set up to run kube-apiserver as a service on each Kubernetes control node.

    You can configure the Kubernetes API server like so:
```
    sudo mkdir -p /var/lib/kubernetes/
    sudo cp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
      service-account-key.pem service-account.pem \
      encryption-config.yaml /var/lib/kubernetes/
```
    Set some environment variables that will be used to create the systemd unit file. Make sure you replace the placeholders with their actual values:
```
    INTERNAL_IP=$(curl http://169.254.169.254/latest/meta-data/local-ipv4)
    CONTROLLER0_IP=<private ip of controller 0>
    CONTROLLER1_IP=<private ip of controller 1>
```
    Generate the kube-apiserver unit file for systemd:
```
    cat << EOF | sudo tee /etc/systemd/system/kube-apiserver.service
    [Unit]
    Description=Kubernetes API Server
    Documentation=https://github.com/kubernetes/kubernetes

    [Service]
    ExecStart=/usr/local/bin/kube-apiserver \\
      --advertise-address=${INTERNAL_IP} \\
      --allow-privileged=true \\
      --apiserver-count=3 \\
      --audit-log-maxage=30 \\
      --audit-log-maxbackup=3 \\
      --audit-log-maxsize=100 \\
      --audit-log-path=/var/log/audit.log \\
      --authorization-mode=Node,RBAC \\
      --bind-address=0.0.0.0 \\
      --client-ca-file=/var/lib/kubernetes/ca.pem \\
      --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
      --enable-swagger-ui=true \\
      --etcd-cafile=/var/lib/kubernetes/ca.pem \\
      --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \\
      --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \\
      --etcd-servers=https://$CONTROLLER0_IP:2379,https://$CONTROLLER1_IP:2379 \\
      --event-ttl=1h \\
      --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
      --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
      --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \\
      --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \\
      --kubelet-https=true \\
      --runtime-config=api/all \\
      --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
      --service-cluster-ip-range=10.32.0.0/24 \\
      --service-node-port-range=30000-32767 \\
      --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \\
      --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \\
      --v=2 \\
      --kubelet-preferred-address-types=InternalIP,InternalDNS,Hostname,ExternalIP,ExternalDNS
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
```
  ### Setting up the Kubernetes Controller Manager
  Now that we have set up kube-apiserver, we are ready to configure kube-controller-manager. This lesson walks you through the process of configuring a systemd service for the Kubernetes Controller Manager. After completing this lesson, you should have the kubeconfig and systemd unit file set up and ready to run the kube-controller-manager service on both of your control nodes.

    You can configure the Kubernetes Controller Manager like so:
    ```
    sudo cp kube-controller-manager.kubeconfig /var/lib/kubernetes/
    ```
    Generate the kube-controller-manager systemd unit file:
    ```
    cat << EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
    [Unit]
    Description=Kubernetes Controller Manager
    Documentation=https://github.com/kubernetes/kubernetes

    [Service]
    ExecStart=/usr/local/bin/kube-controller-manager \\
      --address=0.0.0.0 \\
      --cluster-cidr=10.200.0.0/16 \\
      --cluster-name=kubernetes \\
      --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
      --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
      --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
      --leader-elect=true \\
      --root-ca-file=/var/lib/kubernetes/ca.pem \\
      --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
      --service-cluster-ip-range=10.32.0.0/24 \\
      --use-service-account-credentials=true \\
      --v=2
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
  ### Setting up the Kubernetes Scheduler
Now we are ready to set up the Kubernetes scheduler. This lesson will walk you through the process of configuring the kube-scheduler systemd service. Since this is the last of the three control plane services that need to be set up in this section, this lesson also guides you through through enabling and starting all three services on both control nodes. Finally, this lesson shows you how to verify that your Kubernetes controllers are healthy and working so far. After completing this lesson, you will have a basic, working, Kuberneets control plane distributed across your two control nodes.

You can configure the Kubernetes Sheduler like this.

Copy kube-scheduler.kubeconfig into the proper location:
```
sudo cp kube-scheduler.kubeconfig /var/lib/kubernetes/
```
Generate the kube-scheduler yaml config file.
```
cat << EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```
Create the kube-scheduler systemd unit file:
```
cat << EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
Start and enable all of the services:
```
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
```
It's a good idea to verify that everything is working correctly so far: Make sure all the services are active (running):
```
sudo systemctl status kube-apiserver kube-controller-manager kube-scheduler
```
Use kubectl to check componentstatuses:
```
kubectl get componentstatuses --kubeconfig admin.kubeconfig
```
You should get output that looks like this:
```
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health": "true"}
etcd-1               Healthy   {"health": "true"}
```
  ### Enable HTTP Health Checks for Kube API Service
    Part of Kelsey Hightower's original Kubernetes the Hard Way guide involves setting up an nginx proxy on each controller to provide access to the Kubernetes API /healthz endpoint over http. This lesson explains the reasoning behind the inclusion of that step and guides you through the process of implementing the http /healthz proxy.

    You can set up a basic nginx proxy for the healthz endpoint by first installing nginx"
  ```
    sudo apt-get install -y nginx
  ```
    Create an nginx configuration for the health check proxy:
  ```
    cat > kubernetes.default.svc.cluster.local << EOF
    server {
      listen      80;
      server_name kubernetes.default.svc.cluster.local;

      location /healthz {
        proxy_pass                    https://127.0.0.1:6443/healthz;
        proxy_ssl_trusted_certificate /var/lib/kubernetes/ca.pem;
      }
    }
    EOF
  ```
    Set up the proxy configuration so that it is loaded by nginx:
  ```
    sudo mv kubernetes.default.svc.cluster.local /etc/nginx/sites-available/kubernetes.default.svc.cluster.local
    sudo ln -s /etc/nginx/sites-available/kubernetes.default.svc.cluster.local /etc/nginx/sites-enabled/
    sudo systemctl restart nginx
    sudo systemctl enable nginx
  ```
    You can verify that everything is working like so:
  ```
    curl -H "Host: kubernetes.default.svc.cluster.local" -i http://127.0.0.1/healthz
  ```
    You should receive a 200 OK response.

  ### Set up RBAC for Kubelet Authorization
    One of the necessary steps in setting up a new Kubernetes cluster from scratch is to assign permissions that allow the Kubernetes API to access various functionality within the worker kubelets. This lesson guides you through the process of creating a ClusterRole and binding it to the kubernetes user so that those permissions will be in place. After completing this lesson, your cluster will have the necessary role-based access control configuration to allow the cluster's API to access kubelet functionality such as logs and metrics.
    - RBAC used in k8 to create role and assign permission to diff users. 
    - Make sure that kube api has permission to access the kubelet to perform the necessary action. 

    You can configure RBAC for kubelet authorization with these commands. Note that these commands only need to be run on one control node.

    Create a role with the necessary permissions:

    cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRole
    metadata:
      annotations:
        rbac.authorization.kubernetes.io/autoupdate: "true"
      labels:
        kubernetes.io/bootstrapping: rbac-defaults
      name: system:kube-apiserver-to-kubelet
    rules:
      - apiGroups:
          - ""
        resources:
          - nodes/proxy
          - nodes/stats
          - nodes/log
          - nodes/spec
          - nodes/metrics
        verbs:
          - "*"
    EOF

    Bind the role to the kubernetes user:

    cat << EOF | kubectl apply --kubeconfig admin.kubeconfig -f -
    apiVersion: rbac.authorization.k8s.io/v1beta1
    kind: ClusterRoleBinding
    metadata:
      name: system:kube-apiserver
      namespace: ""
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: system:kube-apiserver-to-kubelet
    subjects:
      - apiGroup: rbac.authorization.k8s.io
        kind: User
        name: kubernetes
    EOF



  ### Setting up a Kube API Frontend Load Balancer
    In order to achieve redundancy for your Kubernetes cluster, you will need to load balance usage of the Kubernetes API across multiple control nodes. In this lesson, you will learn how to create a simple nginx server to perform this balancing. After completing this lesson, you will be able to interact with both control nodes of your kubernetes cluster using the nginx load balancer.

    Here are the commands you can use to set up the nginx load balancer. Run these on the server that you have designated as your load balancer server:
    ```
    sudo apt-get install -y nginx
    sudo systemctl enable nginx
    sudo mkdir -p /etc/nginx/tcpconf.d
    sudo vi /etc/nginx/nginx.conf
    ```
    Add the following to the end of nginx.conf:
    ```
    include /etc/nginx/tcpconf.d/*;

    Set up some environment variables for the lead balancer config file:

    CONTROLLER0_IP=<controller 0 private ip>
    CONTROLLER1_IP=<controller 1 private ip>
    ```
    Create the load balancer nginx config file:
    ```
    cat << EOF | sudo tee /etc/nginx/tcpconf.d/kubernetes.conf
    stream {
        upstream kubernetes {
            server $CONTROLLER0_IP:6443;
            server $CONTROLLER1_IP:6443;
        }

        server {
            listen 6443;
            listen 443;
            proxy_pass kubernetes;
        }
    }
    EOF
    ```
    Reload the nginx configuration:
    ```
    sudo nginx -s reload
    ```
    You can verify that the load balancer is working like so:
    ```
    curl -k https://localhost:6443/version
    ```
    You should get back some json containing version information for your Kubernetes cluster.
 ## Bootstrapping the Kubernetes Worker Nodes
   ###  What are the Kubernetes Worker Nodes?
    Now that we have set up the Kubernetes control plane, we are ready to begin setting up worker nodes. This lesson introduces Kubernetes worker nodes and describes the various components that we will need for installing and configuring them. After completing this lesson, you will have a basic understanding of what worker nodes do and the components that are used to create a worker node.
    - kubelet 
    - kube-proxy 
        Manages the IPtables rule which creates a virtual network (Overlay Network) that manages the network. 
    - container runtime:
     
    You can find more information on Kubernetes the worker nodes in the official documentation:
    https://kubernetes.io/docs/concepts/architecture/
    https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node

   ### Installing Worker Node Binaries
    We are now ready to begin the process of setting up our worker nodes. The first step is to download and install the binary file which we will later use to configure our worker nodes services. In this lesson, we will be downloading and installing the binaries for containerd, kubectl, kubelet, and kube-proxy, as well as other software that they depend on. After completing this lesson, you should have these binaries downloaded and all of the files moved into the correct locations in preparation for configuring the worker node services.

    You can install the worker binaries like so. Run these commands on both worker nodes:
    ```
    sudo apt-get -y install socat conntrack ipset

    wget -q --show-progress --https-only --timestamping \
      https://github.com/kubernetes-incubator/cri-tools/releases/download/v1.0.0-beta.0/crictl-v1.0.0-beta.0-linux-amd64.tar.gz \
      https://storage.googleapis.com/kubernetes-the-hard-way/runsc \
      https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
      https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
      https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
      https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
      https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
      https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet

    sudo mkdir -p \
      /etc/cni/net.d \
      /opt/cni/bin \
      /var/lib/kubelet \
      /var/lib/kube-proxy \
      /var/lib/kubernetes \
      /var/run/kubernetes

    chmod +x kubectl kube-proxy kubelet runc.amd64 runsc

    sudo mv runc.amd64 runc

    sudo mv kubectl kube-proxy kubelet runc runsc /usr/local/bin/

    sudo tar -xvf crictl-v1.0.0-beta.0-linux-amd64.tar.gz -C /usr/local/bin/

    sudo tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/

    sudo tar -xvf containerd-1.1.0.linux-amd64.tar.gz -C /
    ```
   ### Configuring Containerd
    Containerd is the container runtime used to run containers managed by Kubernetes in this course. In this lesson, we will configure a systemd service for containerd on both of our worker node servers. This containerd service will be used to run containerd as a component of each worker node. After completing this lesson, you should have a containerd configured to run as a systemd service on both workers.

    You can configure the containerd service like so. Run these commands on both worker nodes:
    ```
    sudo mkdir -p /etc/containerd/
    ```
    Create the containerd config.toml:
    ```
    cat << EOF | sudo tee /etc/containerd/config.toml
    [plugins]
      [plugins.cri.containerd]
        snapshotter = "overlayfs"
        [plugins.cri.containerd.default_runtime]
          runtime_type = "io.containerd.runtime.v1.linux"
          runtime_engine = "/usr/local/bin/runc"
          runtime_root = ""
        [plugins.cri.containerd.untrusted_workload_runtime]
          runtime_type = "io.containerd.runtime.v1.linux"
          runtime_engine = "/usr/local/bin/runsc"
          runtime_root = "/run/containerd/runsc"
    EOF
    ```
    Create the containerd unit file:
    ```
    cat << EOF | sudo tee /etc/systemd/system/containerd.service
    [Unit]
    Description=containerd container runtime
    Documentation=https://containerd.io
    After=network.target

    [Service]
    ExecStartPre=/sbin/modprobe overlay
    ExecStart=/bin/containerd
    Restart=always
    RestartSec=5
    Delegate=yes
    KillMode=process
    OOMScoreAdjust=-999
    LimitNOFILE=1048576
    LimitNPROC=infinity
    LimitCORE=infinity

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
   ### Configuring Kubelet
    Kubelet is the Kubernetes agent which runs on each worker node. Acting as a middleman between the Kubernetes control plane and the underlying container runtime, it coordinates the running of containers on the worker node. In this lesson, we will configure our systemd service for kubelet. After completing this lesson, you should have a systemd service configured and ready to run on each worker node.

    You can configure the kubelet service like so. Run these commands on both worker nodes.

    Set a HOSTNAME environment variable that will be used to generate your config files. Make sure you set the HOSTNAME appropriately for each worker node:
    ```
    HOSTNAME=$(hostname)
    sudo mv ${HOSTNAME}-key.pem ${HOSTNAME}.pem /var/lib/kubelet/
    sudo mv ${HOSTNAME}.kubeconfig /var/lib/kubelet/kubeconfig
    sudo mv ca.pem /var/lib/kubernetes/
    ```
    Create the kubelet config file:
    ```
    cat << EOF | sudo tee /var/lib/kubelet/kubelet-config.yaml
    kind: KubeletConfiguration
    apiVersion: kubelet.config.k8s.io/v1beta1
    authentication:
      anonymous:
        enabled: false
      webhook:
        enabled: true
      x509:
        clientCAFile: "/var/lib/kubernetes/ca.pem"
    authorization:
      mode: Webhook
    clusterDomain: "cluster.local"
    clusterDNS: 
      - "10.32.0.10"
    runtimeRequestTimeout: "15m"
    tlsCertFile: "/var/lib/kubelet/${HOSTNAME}.pem"
    tlsPrivateKeyFile: "/var/lib/kubelet/${HOSTNAME}-key.pem"
    EOF
    ```
    Create the kubelet unit file:
    ```
    cat << EOF | sudo tee /etc/systemd/system/kubelet.service
    [Unit]
    Description=Kubernetes Kubelet
    Documentation=https://github.com/kubernetes/kubernetes
    After=containerd.service
    Requires=containerd.service

    [Service]
    ExecStart=/usr/local/bin/kubelet \\
      --config=/var/lib/kubelet/kubelet-config.yaml \\
      --container-runtime=remote \\
      --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \\
      --image-pull-progress-deadline=2m \\
      --kubeconfig=/var/lib/kubelet/kubeconfig \\
      --network-plugin=cni \\
      --register-node=true \\
      --v=2 \\
      --hostname-override=${HOSTNAME} \\
      --allow-privileged=true
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
   ### Configuring Kube-Proxy
    Kube-proxy is an important component of each Kubernetes worker node. It is responsible for providing network routing to support Kubernetes networking components. In this lesson, we will configure our kube-proxy systemd service. Since this is the last of the three worker node services that we need to configure, we will also go ahead and start all of our worker node services once we're done. Finally, we will complete some steps to verify that our cluster is set up properly and functioning as expected so far. After completing this lesson, you should have two Kubernetes worker nodes up and running, and they should be able to successfully register themselves with the cluster.

    You can configure the kube-proxy service like so. Run these commands on both worker nodes:
    ```
    sudo mv kube-proxy.kubeconfig /var/lib/kube-proxy/kubeconfig
    ```
    Create the kube-proxy config file:
    ```
    cat << EOF | sudo tee /var/lib/kube-proxy/kube-proxy-config.yaml
    kind: KubeProxyConfiguration
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    clientConnection:
      kubeconfig: "/var/lib/kube-proxy/kubeconfig"
    mode: "iptables"
    clusterCIDR: "10.200.0.0/16"
    EOF
    ```
    Create the kube-proxy unit file:
    ```
    cat << EOF | sudo tee /etc/systemd/system/kube-proxy.service
    [Unit]
    Description=Kubernetes Kube Proxy
    Documentation=https://github.com/kubernetes/kubernetes

    [Service]
    ExecStart=/usr/local/bin/kube-proxy \\
      --config=/var/lib/kube-proxy/kube-proxy-config.yaml
    Restart=on-failure
    RestartSec=5

    [Install]
    WantedBy=multi-user.target
    EOF
    ```
    Now you are ready to start up the worker node services! Run these:
    ```
    sudo systemctl daemon-reload
    sudo systemctl enable containerd kubelet kube-proxy
    sudo systemctl start containerd kubelet kube-proxy
    ```
    Check the status of each service to make sure they are all active (running) on both worker nodes:
    ```
    sudo systemctl status containerd kubelet kube-proxy
    ```
    Finally, verify that both workers have registered themselves with the cluster. Log in to one of your control nodes and run this:
    ```
    kubectl get nodes
    ```
    You should see the hostnames for both worker nodes listed. Note that it is expected for them to be in the NotReady state at this point.
  ## Configuring kubectl for Remote Access
   ### Kubernetes Remote Access and kubectl
   Kubectl is a powerful command-line tool that allows you to manage Kubernetes clusters. In order to manage the cluster from your machine, you will need to configure your local kubectl to connect to the remote cluster. This lesson discusses kubectl and introduces the rationale behind using it to manage the cluster remotely. It also discusses how a local kubectl installation fits into the larger overall cluster architecture. This will provide some background to help you understand the following lesson, which demonstrates how to implement this configuration.

   ### Configuring Kubectl for Remote Access
    There are a few steps to configuring a local kubectl installation for managing a remote cluster. This lesson will guide you through that process. After completing this lesson, you should have a local kubectl installation that is capable of running kubectl commands against your remote Kubernetes cluster.

    In a separate shell, open up an ssh tunnel to port 6443 on your Kubernetes API load balancer:
    ```
    ssh -L 6443:localhost:6443 user@<your Load balancer cloud server public IP>
    ```
    You can configure your local kubectl in your main shell like so. Set KUBERNETES_PUBLIC_ADDRESS to the public IP of your load balancer.
    ```
    cd ~/kthw

    kubectl config set-cluster kubernetes-the-hard-way \
      --certificate-authority=ca.pem \
      --embed-certs=true \
      --server=https://localhost:6443

    kubectl config set-credentials admin \
      --client-certificate=admin.pem \
      --client-key=admin-key.pem

    kubectl config set-context kubernetes-the-hard-way \
      --cluster=kubernetes-the-hard-way \
      --user=admin

    kubectl config use-context kubernetes-the-hard-way
    ```
    Verify that everything is working with:
    ```
    kubectl get pods
    kubectl get nodes
    kubectl version
    ```
  ## Networking
