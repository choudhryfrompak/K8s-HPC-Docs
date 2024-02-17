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
   
## Worker Node Setup

Follow similar steps for setting up worker nodes:

1. Set the hostname and map hostnames to IP addresses.

2. Update server, install necessary packages, and configure containerd.

3. Install Kubernetes components.

4. Disable swap and load `br_netfilter` module.

5. Run the command saved during Kubernetes initialization on the master node to join the worker nodes to the cluster.

## Conclusion

With the master and worker nodes set up, you have a fully functioning Kubernetes cluster ready to deploy and manage applications.
