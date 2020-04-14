## Let's start

### First: Some requirement

On each raspberry pi execute the following commands:

    sudo -i
  
    apt-get update && apt-get install -y iptables arptables ebtables ipvsadm
    update-alternatives --set iptables /usr/sbin/iptables-legacy
    update-alternatives --set ip6tables /usr/sbin/ip6tables-legacy
    update-alternatives --set arptables /usr/sbin/arptables-legacy
    update-alternatives --set ebtables /usr/sbin/ebtables-legacy
  
    apt-get update && apt-get install vim
  
    apt-get update && apt-get install -y apt-transport-https curl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
    cat <<EOF | tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF
    apt-get update
    apt-get install -y kubelet kubeadm kubectl
    apt-mark hold kubelet kubeadm kubectl
  
    apt-get update && apt-get upgrade
  
    reboot
  
### Second: Start your first control plane
  
I use rasp05. Replace the endoint IP by your master IP.
  
    kubeadm config images pull
    kubeadm init --control-plane-endpoint=10.10.50.105
      
The output should look like:

    [init] Using Kubernetes version: vX.Y.Z
    [preflight] Running pre-flight checks
    [preflight] Pulling images required for setting up a Kubernetes cluster
    [preflight] This might take a minute or two, depending on the speed of your internet connection
    [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
    [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [kubelet-start] Activating the kubelet service
    [certs] Using certificateDir folder "/etc/kubernetes/pki"
    [certs] Generating "etcd/ca" certificate and key
    [certs] Generating "etcd/server" certificate and key
    [certs] etcd/server serving cert is signed for DNS names [kubeadm-cp localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
    [certs] Generating "etcd/healthcheck-client" certificate and key
    [certs] Generating "etcd/peer" certificate and key
    [certs] etcd/peer serving cert is signed for DNS names [kubeadm-cp localhost] and IPs [10.138.0.4 127.0.0.1 ::1]
    [certs] Generating "apiserver-etcd-client" certificate and key
    [certs] Generating "ca" certificate and key
    [certs] Generating "apiserver" certificate and key
    [certs] apiserver serving cert is signed for DNS names [kubeadm-cp kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.138.0.4]
    [certs] Generating "apiserver-kubelet-client" certificate and key
    [certs] Generating "front-proxy-ca" certificate and key
    [certs] Generating "front-proxy-client" certificate and key
    [certs] Generating "sa" key and public key
    [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
    [kubeconfig] Writing "admin.conf" kubeconfig file
    [kubeconfig] Writing "kubelet.conf" kubeconfig file
    [kubeconfig] Writing "controller-manager.conf" kubeconfig file
    [kubeconfig] Writing "scheduler.conf" kubeconfig file
    [control-plane] Using manifest folder "/etc/kubernetes/manifests"
    [control-plane] Creating static Pod manifest for "kube-apiserver"
    [control-plane] Creating static Pod manifest for "kube-controller-manager"
    [control-plane] Creating static Pod manifest for "kube-scheduler"
    [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
    [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
    [apiclient] All control plane components are healthy after 31.501735 seconds
    [uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
    [kubelet] Creating a ConfigMap "kubelet-config-X.Y" in namespace kube-system with the configuration for the kubelets in the cluster
    [patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm-cp" as an annotation
    [mark-control-plane] Marking the node kubeadm-cp as control-plane by adding the label "node-role.kubernetes.io/master=''"
    [mark-control-plane] Marking the node kubeadm-cp as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
    [bootstrap-token] Using token: <token>
    [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
    [bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
    [bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
    [bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
    [bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
    [addons] Applied essential addon: CoreDNS
    [addons] Applied essential addon: kube-proxy

    Your Kubernetes control-plane has initialized successfully!

    To start using your cluster, you need to run the following as a regular user:

      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config

    You should now deploy a Pod network to the cluster.
    Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at: /docs/concepts/cluster-administration/addons/

    You can now join any number of control-plane nodes by copying certificate authorities
    and service account keys on each node and then running the following as root:

    kubeadm join 10.10.50.105:6443 --token 8kihxh.ejtse08m7dsl6vhs --discovery-token-ca-cert-hash sha256:4257c3d9e57a99d3778a2a89bdd954d40289d358d6de133cb28a9a3792cbbc09 --control-plane 

    Then you can join any number of worker nodes by running the following on each as root:

    kubeadm join 10.10.50.105:6443 --token 8kihxh.ejtse08m7dsl6vhs --discovery-token-ca-cert-hash sha256:4257c3d9e57a99d3778a2a89bdd954d40289d358d6de133cb28a9a3792cbbc09

To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:

    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config

Now, install a pod network. I choose Weave Net.

    kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
    
### Third: other control plane and nodes

#### Add other control plane

I choose to add rasp00.

On rasp05 create a script certificate.sh:

    USER=pirate # customizable
    CONTROL_PLANE_IPS="10.10.50.100"
    for host in ${CONTROL_PLANE_IPS}; do
        scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
        scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
        scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
        scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
        scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
        scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
        scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host:etcd-ca.crt
        # Quote this line if you are using external etcd
        scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host:etcd-ca.key
    done

Add the right to execute and execute:

    chmod +x certificate.sh
    ./certficate.sh

On rasp00 create a script certificate.sh:

    USER=pirate # customizable
    mkdir -p /etc/kubernetes/pki/etcd
    mv /home/${USER}/ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/ca.key /etc/kubernetes/pki/
    mv /home/${USER}/sa.pub /etc/kubernetes/pki/
    mv /home/${USER}/sa.key /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
    mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
    # Quote this line if you are using external etcd
    mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key

Add the right to execute and execute:

    chmod +x certificate.sh
    ./certficate.sh

Now, you can join rasp00 with follow this command:

    kubeadm join 10.10.50.105:6443 --token 8kihxh.ejtse08m7dsl6vhs --discovery-token-ca-cert-hash sha256:4257c3d9e57a99d3778a2a89bdd954d40289d358d6de133cb28a9a3792cbbc09 --control-plane 
    
#### Add nodes

I choose rasp01, rasp06 and rasp07.

You can join the rasp with follow this command:

    kubeadm join 10.10.50.105:6443 --token 8kihxh.ejtse08m7dsl6vhs --discovery-token-ca-cert-hash sha256:4257c3d9e57a99d3778a2a89bdd954d40289d358d6de133cb28a9a3792cbbc09

Now, you can see all nodes by executing the following command on one of your control plane:

    kubectl get nodes


      
      
      
      
      
