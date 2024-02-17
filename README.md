# K8s-HPC-Docs
This Documentation is about setting up a kubernetes cluster based on K8s version of kubernetes. This clusterwill be deployment ready for GPU Workloads and all other workloads using the core Runtime.
For the process we need At Least:
1 Master node/ Control plane
1 worker Node
1 Dashboard(Raancher) Node

These all nodes must have static IP Adresses. You can setup a static IP by Following this Guide
# Setting up a Kubernetes Cluster

## Master Node Setup

After SSH into the master node instance, follow these steps:

1. Change the hostname to `k8s-master`:
    ```bash
    sudo su
    hostnamectl set-hostname k8s-master
    ```

2. Map hostnames to IP addresses by adding entries to `/etc/hosts`:
    ```bash
    cat << EOF >> /etc/hosts
    <insert master node ip> k8s-master
    <insert worker 1 ip> k8s-worker1
    <insert worker 2 ip> k8s-worker2
    EOF
    ```

3. Exit the root and the server, then sign back in.

4. Update the server and install necessary packages:
    ```bash
    sudo apt-get update -y
    sudo apt install -y apt-transport-https curl vim git
    ```

5. Install containerd:
    ```bash
    # Install containerd
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y containerd.io

    # Configure containerd
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    ```

6. Set `SystemdCgroup` to `true` within `config.toml`, then restart containerd daemon and enable it to start automatically at boot time.
    ```bash
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```

7. Install Kubernetes components:
    ```bash
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
    sudo apt-add-repository -y "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    sudo apt install -y kubeadm kubelet kubectl kubernetes-cni
    ```

8. Disable swap and load `br_netfilter` module in the Linux kernel.
    ```bash
    sudo swapoff -a
    sudo modprobe br_netfilter
    ```

9. Initialize Kubernetes cluster:
    ```bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```

10. Allow `kubectl` to interact with the cluster:
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    export KUBECONFIG=/etc/kubernetes/admin.conf
    ```

11. Install CNI Flannel for networking:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
    ```

## Worker Node Setup

Follow similar steps for setting up worker nodes:

1. Set the hostname and map hostnames to IP addresses.

2. Update server, install necessary packages, and configure containerd.

3. Install Kubernetes components.

4. Disable swap and load `br_netfilter` module.

5. Run the command saved during Kubernetes initialization on the master node to join the worker nodes to the cluster.

## Conclusion

With the master and worker nodes set up, you have a fully functioning Kubernetes cluster ready to deploy and manage applications.
