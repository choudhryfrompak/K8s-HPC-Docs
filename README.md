# K8s-HPC-Docs
This Documentation is about setting up a kubernetes cluster based on K8s version of kubernetes. This cluster will be deployment ready for GPU Workloads and all other workloads using the core Runtime.
For the process we need At Least:
- 1 Master node/ Control plane
- 1 worker Node
- 1 Dashboard(Raancher) Node

These all nodes must have static IP Adresses. You can setup a static IP by Following this Guide

# Set a Static IP Using the Command Line
In this section, we will explore all the steps in detail needed to configure a static IP.

## Step 1: Launch the terminal
You can launch the terminal using the shortcut Ctrl+ Shift+t.

## Step 2: Note information about the current network
We will need our current network details such as the current assigned IP, subnet mask, and the network adapter name so that we can apply the necessary changes in the configurations.

Use the command below to find details of the available adapters and the respective IP information.

```bash 
ip a
```
The output will look something like this:

![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/324cf503-d05d-474c-82ac-96f2c1bed1cb)

For my network, the current adapter is `eth0`. It could be different for your system

Note the current network adapter name
As my current adapter is `eth0`, the below details are relevant.
```bash
6: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:df:c3:ad brd ff:ff:ff:ff:ff:ff
    inet 172.23.199.129/20 brd 172.23.207.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fedf:c3ad/64 scope link
       valid_lft forever preferred_lft forever
```
It is worth noting that the current IP `172.23.199.129` is dynamically assigned. It has 20 bits reserved for the netmask. The broadcast address is `172.23.207.255`.

### Note the subnet
We can find the subnet mask details using the command below:
```bash
ifconfig -a
```
Select the output against your adapter and read it carefully.

![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/a5bf09bd-1a1e-4c5b-8a3d-072ba4a42d4d)

IP is `172.23.199.129` and subnet mask is `255.255.240.0`
Based on the class and subnet mask, the usable host IP range for my network is: `172.23.192.1 - 172.23.207.254`.

## Step 3: Make configuration changes
Netplan is the default network management tool for the latest Ubuntu versions. Configuration files for Netplan are written using YAML and end with the extension `.yaml`.

### Note:
Be careful about spaces in the configuration file as they are part of the syntax. Without proper indentation, the file won't be read properly.

Go to the netplan directory located at `/etc/netplan`.
`ls` into the `/etc/netplan` directory.

If you do not see any files, you can create one. The name could be anything, but by convention, it should start with a number like `01-` and end with `.yaml`. The number sets the priority if you have more than one configuration file.

I'll create a file named `01-network-manager-all.yaml.`

Let's add these lines to the file. We'll build the file step by step.

```bash
network:
 version: 2
```

The top-level node in a Netplan configuration file is a network: mapping that contains version: 2 (means that it is using network definition version 2).

Next, we'll add a renderer, that controls the overall network. The renderer is systemd-networkd by default, but we'll set it to `NetworkManager`.

Now, our file looks like this:
```bash
network:
 version: 2
 renderer: NetworkManager
```
Next, we'll add ethernets and refer to the network adapter name we looked for earlier in `step#2`. Other device types supported are `modems:, wifis:, or bridges:`.
```bash
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
```

As we are setting a static IP and we do not want to dynamically assign an IP to this network adapter, we'll set `dhcp4 to no`.
```bash
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
```
Now we'll specify the specific static IP we noted in `step #2` depending on our subnet and the usable `IP range`. It was `172.23.207.254`.

Next, we'll specify the `gateway`, which is the router or network device that assigns the IP addresses. Mine is on `192.168.1.1`.
```bash
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     addresses: [172.23.207.254/20]
     gateway4: 192.168.1.1
```
Next, we'll define `nameservers`. This is where you define a DNS server or a second DNS server. Here the first value is  `8.8.8.8` which is Google's primary DNS server and the second value is `8.8.8.4` which is Google's secondary DNS server. These values can vary depending on your requirements.
```bash
network:
 version: 2
 renderer: NetworkManager
 ethernets:
   eth0:
     dhcp4: no
     addresses: [172.23.207.254/20]
     gateway4: 192.168.1.1
     nameservers:
         addresses: [8.8.8.8,8.8.8.4]
```
## Step 4: Apply and test the changes
We can test the changes first before permanently applying them using this command:
```bash
sudo netplan try
```
If there are no errors, it will ask if you want to apply these settings.

Now, finally, test the changes with the command `ip a` and you'll see that the `static IP` has been applied.

![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/17cc1cf3-d115-4f83-8480-3fb4d34573b3)

# Set a Static IP Using the GUI
It is very easy to set a static IP through the Ubuntu GUI/ Desktop. Here are the steps:

- Search for settings.
- Click on either Network or Wi-Fi tab, depending on the interface you would like to modify.
- To open the interface settings, click on the gear icon next to the interface name.
- Select “Manual” in the IPV4 tab and enter your static IP address, Netmask and Gateway.
- Click on the Apply button.
![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/8c02a4a2-8fce-4706-babb-0142ec44eec5)


Verify by using the command `ip a`

![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/0bc6ee05-2847-4836-858b-efa36a108a64)

Repeat this process on all Nodes that don't have a static IP

# Setting up a Kubernetes Cluster
Starting the Setup of The Kubernetes Cluster:
- I would recommend setting up `SSH` on all nodes. Because it will make it easy to setup.
### The Guide to enable `ssh` is here:

## Enabling SSH
- in case you have to enable ssh access on all computers. you can do this by following the processs below
- open the terminal and run:

```bash
  sudo apt-get update
```
let the packages update. after that run:

```bash
  sudo apt install openssh-server
```
This command will install openssh-server on your machine

- Enable ssh server on Ubuntu, run: 
```bash
  sudo systemctl enable ssh
```
- By default, firewall will block ssh access. Therefore, you must enable ufw and open ssh port
Open ssh tcp port 22 using ufw firewall, run:

```bash
  sudo ufw allow ssh
```
- Now to check status of ssh-server run:
```bash
    sudo systemctl status ssh
```
- Your ssh is now enabled:

- Now to get your public ip adress run:

```bash
    hostname -I
```
- note your IP ADDRESS and machine name and make a command like:

```bash
    ssh <username>@<ip-address>
```
- Replace <username> with your machine name and <ip-address> with your machine ip. the command will look like:

```bash
    ssh k8s-master@192.168.xx.x
```
- Now you can use your laptop That's on the same network to `ssh` into all the machines and install kubernetes on them.

# Master Node Setup

After SSH into the master node instance, follow these steps:

1. Change the hostname to `k8s-master`:
    ```bash
    sudo su
    hostnamectl set-hostname k8s-master
    ```

2. Map hostnames to IP addresses by adding entries to `/etc/hosts`:
3. you can do this by modifying & running this command:
    ```bash
    cat << EOF >> /etc/hosts
    <insert master node ip> k8s-master
    <insert worker 1 ip> k8s-worker1
    <insert worker 2 ip> k8s-worker2
    EOF
    ```

4. Exit the root and the server, then ssh back in.

5. Update the server and install necessary packages:
    ```bash
    sudo apt-get update -y
    sudo apt install -y apt-transport-https curl vim git
    ```
Now to make a container Runtime for our pods we need to install Cntainerd. Later on we will configure it with nvidia GPU device plugin to make GPUs work in our pods.
6. Install containerd by running this:
    ```bash
    # Install containerd
    sudo mkdir -p /etc/apt/keyrings
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt-get update -y
    sudo apt-get install -y containerd.io ```


```bash
    # Configure containerd
    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
```

7. Set `SystemdCgroup` to `true` within `config.toml`, then restart containerd daemon and enable it to start automatically at boot time.
    ```bash
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```

   - After this, We can proceed to Kubernetes components installation

8. Install Kubernetes components:
    ```bash
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
    sudo apt-add-repository -y "deb http://apt.kubernetes.io/ kubernetes-xenial main"
    sudo apt install -y kubeadm kubelet kubectl kubernetes-cni
    ```

9. Disable swap and load `br_netfilter` module in the Linux kernel.
    ```bash
    sudo swapoff -a
    #check if a swap entry exists and remove it if it does
    sudo nano /etc/fstab
    sudo modprobe br_netfilter
    ```

10. Initialize Kubernetes cluster:
    ```bash
    sudo kubeadm init --pod-network-cidr=10.244.0.0/16
    ```

11. Allow `kubectl` to interact with the cluster:
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    export KUBECONFIG=/etc/kubernetes/admin.conf
    ```
You will also get a statement like the one below:
```bash
#Command to run on worker nodes
kubeadm join <control-plane-ip>:6443 --token <token> \
 --discovery-token-ca-cert-hash <hash>
```
###You should SAVE THIS JOIN COMMAND to run on worker nodes later.

12. Install CNI Flannel for networking:
    ```bash
    kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/v0.20.2/Documentation/kube-flannel.yml
    ```
- Now at this point the Master node is ready.
  ## NOTE
For some weird reason we have to add the master node to the rancher dashboard first then after that we can join worker nodes with the Master.

## Setting Up Rancher

- For a proper Graphical Dashboard monitoring and management system we are installing rancher.

- The whole proccess of installiing rancher will be carried on the dasshboard node.

- Once you have `SSH ACCES` of the dashboard node like we did on other machines in start and get its `IP ADDRESS`

- Access this node from any other computer lke your laptop powershell to check whether it is publically available or not by using `ssh username@ip_address`

- After accessing it we can now start setting up rancher.

## installing docker-engine
- we need  to install docker image to containerize rancher image

- we can INSTALL the compaitable version suggested by rancher by running this command 

```bashrc
    curl https://releases.rancher.com/install-docker/20.10.sh | sh
```

- Note that the following sysctl setting must be applied:

```net.bridge.bridge-nf-call-iptables=1```

- To confirm whether docker is properly installed run ```docker``` 

## containerizing rancher image:
- We need to make rancher data presit accross reboots.
- Rancher by default stores its data in ```/var/lib/rancher```
- Lets create a space to save data into our specified directory.
- We will make one by Running:
```mkdir /kubernetes/rancher/volume```
```cd /kubeernetes/rancher```
- Now as the docker is properly installed and our volume folder is ready we can proceed by running a command that i have set up to make the installation easy. 

Run:
```bashrc
docker run -d --name rancher-server -v ${PWD}/volume:/var/lib/rancher --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:latest
```
- After running this wait for few minutes to let the rancher server start

- Then go to a browser on any computer on the same network and type the `ip address` of your `Dashboard node`  the port 443 like this:
```bashrc
https://<ip_address>:443
```
Now you will be prompted with this screen.
![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/401a70be-cfd0-42ce-aed8-7b4d4effc678)

## Unlocking Rancher Dashboard

- By default the rancher dashboard is locked the default password is somewhere in the logs of docker container and we need to pull it out for that we will go through some commands
- Go back to terminal of dashboard node
- we will need the container id on which rancher is spinning up. to get that, run: 
```bashrc
docker ps

```

- Now we got the container id run this command:
```bashrc
docker logs <container-id> 2>&1 |grep "Bootstrap Password:"
```
- Replace the ```<container-id>``` with the id of your container. when you run this command this will grab the bootstrap password.

- Now again open the browser and enter that password 
- Now it will ask you to change the password. set your desired password & proceed
- Rancher is now properly installed on your dashboard node and is accessible on the network.

- the next step is to import our main cluster into it.

## importing main cluster to Rancher

- As you are now on the homepage of rancher dashboard you will see a icon on top right that says `import existing` click on that and name your cluster and proceed. no need to mess with any other setting.
![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/2eb06586-f481-4373-968c-7bd65f95e7f3)

- Now it wwill show you some configuration command that starts with `curl -sfl` we will need the command with `-insecure` tag because we dont have ssl certificate don't worry we really dont need that
- the command i am talking about maybe it is last among those three commands. 
- copy that command and open terminal of your master node and don't forget enabling root access by `sudo su -` after that paste and run the command you copied. wait for its completion.

- NOW Go back to browser , you will see that your cluster is succesfully added showing all the resources and ready for containerizing pods.
   ### Now we have our master node added to the RANCHER Dashboard, we can move to adding our Workers to our cluster. But before that we need to install kubernetes on All worker nodes.
  
## Worker Node Setup

Like with the K8s Master, the first step is to set the hostnames. First, map the hostnames to IP addresses:
```bash
#become root
sudo su
```
```bash
#set hostname
hostnamectl set-hostname k8s-worker<number of worker node>
```
```bash
#Maps hostnames to IP addresses
cat << EOF >> /etc/hosts
<insert K8s Masternode ip> k8s-control
<insert worker 1 ip> k8s-worker1
<insert worker 2 ip> k8s-worker2
EOF
```
```bash
#exit root
exit
```
```bash
#exit the server and then sign back in
exit
```

Now completing pre-requisites:
```bash
#update server and install apt-transport-https and curl
sudo apt-get update -y
sudo apt install -y apt-transport-https curl
```
```bash
#Install containerd 
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install -y containerd.io
```
```bash
#Configure containerd
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```
```bash
#set SystemdCgroup = true within configs.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```
```bash
#Restart containerd daemon
sudo systemctl restart containerd
```
```bash
#Enable containerd to start automatically at boot time
sudo systemctl enable containerd
```
Now initializing kubernetes installation:
```bash
#install kubeadm, kubectl, kubelet,and kubernetes-cni 
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add
sudo apt-add-repository -y "deb http://apt.kubernetes.io/ kubernetes-xenial main"
sudo apt install -y kubeadm kubelet kubectl kubernetes-cni
```
```bash
#disable swap
sudo swapoff -a
```
```bahs
#load the br_netfilter module in the Linux kernel
sudo modprobe br_netfilter
```
now you'll need to have your hands on the keyboard to use nano and enter/exit root:
```bash
#check if a swap entry exists and remove it if it does
sudo nano /etc/fstab
```
```bash
#enable ip-forwarding 
sudo su 
echo 1 > /proc/sys/net/ipv4/ip_forward
exit
```

### Finally, you'll want to run the command that we saved when running Kubernetes on the Control Plane:
```bash
#Command to run on worker nodes
kubeadm join <control-plane-ip>:6443 --token <token> \
 --discovery-token-ca-cert-hash <hash>
```
check If the joins have been done correctly on both worker nodes,by running:
```bash
#in root of master node
kubectl get nodes
```
On the Dashboard it looks like
![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/ea61c096-0cb0-4f98-9032-0a534c50370a)

after clicking on the cluster:
![image](https://github.com/choudhryfrompak/K8s-HPC-Docs/assets/129526340/d8e96b7c-24cb-48ac-8ce2-ac2aa5c85d56)


# Preparing cluster for GPU Workloads:

We have to setup this cluster for gpu workloads so to make it able to use the core container runtime to use pods we have to do following things:
## Pre-requisites:
The only pre-requisite is that you must have Nvidia drivers installed. Then we can initialize the setup by:

- Installing Nvidia Container Toolkit on all worker nodes.

Installing Nvidia container toolkit with Apt
- Configure the production repository:
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
- Optionally, configure the repository to use experimental packages:

```bash
sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
Update the packages list from the repository:
```
```bash
sudo apt-get update
```
Install the NVIDIA Container Toolkit packages:
```bash
sudo apt-get install -y nvidia-container-toolkit
```
## Configuring containerd (for Kubernetes)
Configure the container runtime by using the nvidia-ctk command:
```bash
sudo nvidia-ctk runtime configure --runtime=containerd
```
The nvidia-ctk command modifies the /etc/containerd/config.toml file on the host. The file is updated so that containerd can use the NVIDIA Container Runtime.

### Restart containerd:
```bash
sudo systemctl restart containerd
```

### Configure containerd
When running kubernetes with containerd, edit the config file which is usually present at /etc/containerd/config.toml to set up nvidia-container-runtime as the default low-level runtime:
```bash
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      default_runtime_name = "nvidia"

      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia]
          privileged_without_host_devices = false
          runtime_engine = ""
          runtime_root = ""
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.nvidia.options]
            BinaryName = "/usr/bin/nvidia-container-runtime"
```
And then restart containerd:

```bash
sudo systemctl restart containerd
```
- Repeat this on all workers.

  ## Deploying Daemonset for GPU PLUGIN:
  Go in the terminal of Master node and become root
  ```bash
  sudo su -
  ```
  Now to deploy Daemonset run:
  ```bash
  kubectl create -f https://raw.githubusercontent.com/NVIDIA/k8s-device-plugin/v0.14.4/nvidia-device-plugin.yml
  ```
  - This will deploy a daemonset that will handle the gpu workloads.
  After this
```bash
kubectl run podgpu --image=bilal77511/tljh:v1 --restart=Never --overrides='
{
  "apiVersion": "v1",
  "spec": {
    "containers": [
      {
        "name": "podgpu",
        "image": "bilal77511/tljh:v1",
        "imagePullPolicy": "Always",
        "resources": {
          "limits": {
            "nvidia.com/gpu": "1"
          }
        },
        "securityContext": {
          "privileged": true
        },
        "volumeMounts": [
          {
            "mountPath": "/var/run/secrets/kubernetes.io/serviceaccount",
            "name": "kube-api-access-626dl",
            "readOnly": true
          }
        ]
      }
    ],
    "dnsConfig": {
      "nameservers": [
        "8.8.8.8"
      ]
    },
    "dnsPolicy": "ClusterFirst",
    "enableServiceLinks": true,
    "volumes": [
      {
        "name": "kube-api-access-626dl",
        "projected": {
          "defaultMode": 420,
          "sources": [
            {
              "serviceAccountToken": {
                "expirationSeconds": 3607,
                "path": "token"
              }
            },
            {
              "configMap": {
                "name": "kube-root-ca.crt",
                "items": [
                  {
                    "key": "ca.crt",
                    "path": "ca.crt"
                  }
                ]
              }
            },
            {
              "downwardAPI": {
                "items": [
                  {
                    "fieldRef": {
                      "apiVersion": "v1",
                      "fieldPath": "metadata.namespace"
                    },
                    "path": "namespace"
                  }
                ]
              }
            }
          ]
        }
      }
    ]
  }
}'

```

- Now Execute `/bin/bash` shell into the pod by running
```bash
kubectl exec -it podgpu /bin/bash
```
- Now you can test whether GPUs are working or not by running:
```bash
nvidia-smi
```

## Conclusion

With the master and worker nodes set up, you have a fully functioning Kubernetes cluster ready to deploy and manage applications.




