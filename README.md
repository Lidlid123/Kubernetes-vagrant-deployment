# Kubernetes-vagrant-deployment

1. First, make sure you have the Vagrantfile -> clone the repo .

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
   
   4. Forward IPv4 and let iptables see bridged traffic by executing the following instructions:
   
      ```
      cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
      overlay
      br_netfilter
      EOF

      sudo modprobe overlay
      sudo modprobe br_netfilter

      # sysctl params required by setup, params persist across reboots
      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.bridge.bridge-nf-call-iptables  = 1
      net.bridge.bridge-nf-call-ip6tables = 1
      net.ipv4.ip_forward                 = 1
      EOF

      # Apply sysctl params without reboot
      sudo sysctl --system
      
      # Verify that the br_netfilter, overlay modules are loaded by running the following commands:
      
      lsmod | grep br_netfilter
      lsmod | grep overlay
      
      # Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config by running the following command:
      
      sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
      ```
   
   5. Install Docker engine - container runtime (go to the Docker documentation and just install Docker).
   
   6. Install Kubernetes components by running the following commands:
   
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

Is there anything else I can help with? 😊
