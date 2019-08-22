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

### Step 1. Provisioning the Certificate Authority. 
 ----------------------------------------------
In order to generate the certificates needed by Kubernetes, you must first provision a certificate authority. This lesson will guide you through the process of provisioning a new certificate authority for your Kubernetes cluster. After completing this lesson, you should have a certificate authority, which consists of two files: ca-key.pem and ca.pem.

Here are the commands used in the demo:

cd ~/
mkdir kthw
cd kthw/

UPDATE: cfssljson and cfssl will need to be installed. To install, complete the following commands:

sudo curl -s -L -o /bin/cfssl https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
sudo curl -s -L -o /bin/cfssljson https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
sudo curl -s -L -o /bin/cfssl-certinfo https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
sudo chmod +x /bin/cfssl*

Use this command to generate the certificate authority. Include the opening and closing curly braces to run this entire block as a single command.

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

ca-key.pem is the private key 
ca.pem is the public CA that will be moved to each and every components to validate the TLS certificates. 

### Step 2. Generating Client Certificates. 
 --------------------------------------
  Several Client cetiticates needed to be generated. Now that you have provisioned a certificate authority for the Kubernetes cluster, you are ready to begin generating certificates. The first set of certificates you will need to generate consists of the client certificates used by various Kubernetes components. In this lesson, we will generate the following client certificates: admin, kubelet (one for each worker node), kube-controller-manager, kube-proxy, and kube-scheduler. After completing this lesson, you will have the client certificate files which you will need later to set up the cluster.

Here are the commands used in the demo. The command blocks surrounded by curly braces can be entered as a single command:
cd ~/kthw
Admin Client certificate:
-------------------------
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

#### Kubelet Client certificates:
--------------------------------
 These are just a little more conmplicated. Because these certifate is signed for specific host (Node). Kubelet Client certificates. Be sure to enter your actual cloud server values for all four of the variables at the top:

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
#### Controller Manager Client certificate:
------------------------------------------
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

#### Kube Proxy Client certificate:
----------------------------------

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

#### Kube Scheduler Client Certificate:
--------------------------------------

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


 ### Step 3. Generating the Kubernetes API Server Certificate. 
 ---------------------------------------------------------
 We have generated all of the the client certificates our Kubernetes clister will need, but we also need a server certificate for the Kubernetes API. In this lesson, we will generate one, signed with all of the hostnames and IPs that may be used later in order to access the Kubernetes API. After completing this lesson, you will have a Kubernetes API server certificate in the form of two files called kubernetes-key.pem and kubernetes.pem.

Here are the commands used in the demo. Be sure to replace all the placeholder values in CERT_HOSTNAME with their real values from your cloud servers:

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

 ### Step 4. Generating the Service Account Key Pair
 -----------------------------------------------
 Kubernetes provides the ability for service accounts to authenticate using tokens. It uses a key-pair to provide signatures for those tokens. In this lesson, we will generate a certificate that will be used as that key-pair. After completing this lesson, you will have a certificate ready to be used as a service account key-pair in the form of two files: service-account-key.pem and service-account.pem. Used to authenticate with the service account created in the Kubernertes. 

Here are the commands used in the demo:

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

### Step 5. Distributing the Certificate Files
------------------------------------------
Now that all of the necessary certificates have been generated, we need to move the files onto the appropriate servers. In this lesson, we will copy the necessary certificate files to each of our cloud servers. After completing this lesson, your controller and worker nodes should each have the certificate files which they need.

Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.

Move certificate files to the worker nodes:

scp ca.pem <worker 1 hostname>-key.pem <worker 1 hostname>.pem user@<worker 1 public IP>:~/
scp ca.pem <worker 2 hostname>-key.pem <worker 2 hostname>.pem user@<worker 2 public IP>:~/

Move certificate files to the controller nodes:

scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 1 public IP>:~/
scp ca.pem ca-key.pem kubernetes-key.pem kubernetes.pem \
    service-account-key.pem service-account.pem user@<controller 2 public IP>:~/

### Generating Kubeconfig 
 ---------------------
 The next step in building a Kubernetes cluster the hard way is to generate kubeconfigs which will be used by the various services that will make up the cluster. In this lesson, we will generate these kubeconfigs. After completing this lesson, you should have a set of kubeconfigs which you will need later in order to configure the Kubernetes cluster.

Here are the commands used in the demo. Be sure to replace the placeholders with actual values from your cloud servers.

Create an environment variable to store the address of the Kubernetes API, and set it to the private IP of your load balancer cloud server:

KUBERNETES_ADDRESS=<load balancer private ip>

#### Generate a kubelet kubeconfig for each worker node:
----------------------------------------------------

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

#### Generate a kube-proxy kubeconfig:
---------------------------------
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

#### Generate a kube-controller-manager kubeconfig:
----------------------------------------------
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

#### Generate a kube-scheduler kubeconfig:
-------------------------------------
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

#### SGenerate an admin kubeconfig:
-----------------------------
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

  - This creates the necessary configurations for the system pods and nodes. This will help to setup the kubernetes cluster. 


  Now that we have generated the kubeconfig files that we will need in order to configure our Kubernetes cluster, we need to make sure that each cloud server has a copy of the kubeconfig files that it will need. In this lesson, we will distribute the kubeconfig files to each of the worker and controller nodes so that they will be in place for future lessons. After completing this lesson, each of your worker and controller nodes should have a copy of the kubeconfig files it needs.

Here are the commands used in the demo. Be sure to replace the placeholders with the actual values from from your cloud servers.

#### Move kubeconfig files to the worker nodes:
-----------------------------------------------
scp <worker 1 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 1 public IP>:~/
scp <worker 2 hostname>.kubeconfig kube-proxy.kubeconfig user@<worker 2 public IP>:~/

#### Move kubeconfig files to the controller nodes:
---------------------------------------------------
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 1 public IP>:~/
scp admin.kubeconfig kube-controller-manager.kubeconfig kube-scheduler.kubeconfig user@<controller 2 public IP>:~/

### Generating the Data Encryption Config and Key
-------------------------------------------------
  > Kubernetes allows to encrypts the secrets at rest. Meaning it encrypts the data when its in rest. 
  > This will generate the encryption key and put in the config file, and this will be the kubernetes data encryption below: 
   Ref: https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/
   One important security practice is to ensure that sensitive data is never stored in plain text. Kubernetes offers the ability to encrypt sensitive data when it is stored. However, in order to use this feature it is necessary to provide Kubernetes with a data encrpytion config containing an encryption key. This lesson briefly discusses what the data encrpytion config is and why it is needed. This will provide you with some background knowledge as you proceed to create a data encrpytion config for your cluster. After completing this lesson, you will have a basic understanding of what a data encrpytion config is and what it is used for.

 #### Generate the Kubernetes Data encrpytion config file containing the encrpytion key:

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

Copy the file to both controller servers:

scp encryption-config.yaml user@<controller 1 public ip>:~/
scp encryption-config.yaml user@<controller 2 public ip>:~/
