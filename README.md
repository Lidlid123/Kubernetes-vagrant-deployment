# Kubernetes-vagrant-deployment


1. First, make sure you have the Vagrant file that you uploaded to deploy all the nodes in the cluster.

2. After successfully deploying the nodes, you can initiate the Kubernetes deployment for production/development (vanilla deployment).

3. On all nodes, perform the following steps:

   1. Disable `ufw` by running the command `ufw disable`.
   
   2. Disable swap by running the command `swapoff -a; sed -i '/swap/d' /etc/fstab`.
   
   3. Enable bridged traffic to iptables by running the following commands:
   
      ```
      cat >>/etc/sysctl.d/kubernetes.conf<<EOF
      net.bridge.bridge-nf-call-ip6tables = 1
      net.bridge.bridge-nf-call-iptables = 1
      EOF
      sysctl --system
      ```
   
   4. Install Docker engine - container runtime (go to the Docker documentation and just install Docker).
   
   5. Install Kubernetes components by running the following commands:
   
      ```
      sudo apt-get update
      sudo apt-get install -y apt-transport-https ca-certificates curl
      curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
      sudo apt-get update
      sudo apt-get install -y kubelet kubeadm kubectl
      sudo apt-mark hold kubelet kubeadm kubectl
      ```

4. On the master node, perform the following steps:

   1. Initialize the Kubernetes cluster by running the command `kubeadm init --apiserver-advertise-address=172.16.16.100 --pod-network-cidr=192.168.0.0/16`. If you have any preflight errors, check them via Google GPT robot.
   
   2. Set up `kubectl` to use the admin configuration by running the following commands:
   
      ```
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      ```
   
   3. Deploy Calico network by running the command `kubectl --kubeconfig=/etc/kubernetes/admin.conf create -f https://docs.projectcalico.org/v3.14/manifests/calico.yaml`.
   
   4. Generate a join command for worker nodes by running the command `kubeadm token create --print-join-command`.

5. On each worker node, run the join command generated in the previous step from the master server.

That's it! You should now have a Kubernetes cluster up and running with Calico as the network plugin.

Have fun! 😊
