# Multi Master cluster setup using Kubeadm

### What is Kubeadm ? 

Kubeadm is a tool built to provide kubeadm init and kubeadm join as best-practice “fast paths” for creating Kubernetes clusters. kubeadm performs the actions necessary to get a minimum viable cluster up and running. By design, it cares only about bootstrapping, not about provisioning machines. With kubeadm, your cluster should pass Kubernetes Conformance tests. You can find more details about the conformance tests at https://github.com/cncf/k8s-conformance

### Pre-requisite 

For this demo, we will use 2 master and 2 worker node to create a multi master kubernetes cluster using kubeadm installation tool. Below are the pre-requisite requirements for the installation:

* 2 machines for master, ubuntu 16.04+, 2 CPU, 2 GB RAM, 10 GB storage
* 2 machines for worker, ubuntu 16.04+, 1 CPU, 2 GB RAM, 10 GB storage
* 1 machine for loadbalancer, ubuntu 16.04+, 1 CPU, 2 GB RAM, 10 GB storage
* All machines must be accessible on the network. For cloud users - single VPC for all machines 
* sudo privilege 
* ssh access from loadbalancer node to all machines (master & worker). 
* ssh access can be given to any account. ssh through root is not mandatory

Note that we will not cover ssh setup between loadbalancer. 

Below are my virtual machines on GCP - region - us-west1

![image](https://user-images.githubusercontent.com/44743158/68572873-81edcf80-048c-11ea-968a-1de67925696b.png)

---

## Setting up loadbalancer

At this point we will work only on the **loadbalancer** node. In order to setup loadbalancer, you can leverage any loadbalancer utility of your choice. If you feel like using cloud based TCP loadbalancer, feel free to do so. There is no restriction regarding the tool you want to use for this purpose. For our demo, we will use HAPROXY as the primary loadbalancer. 

### What are we loadbalancing ?

We have 2 master nodes. Which means the user can connect to either of the 2 api-servers. The loadbalancer will be used to loadbalance between the 2 api-servers. 

Now that we have some information about what we are trying to achieve, we can now start configuring our loadbalancer node. 

* Login to the loadbalancer node 

* Switch as root - ` sudo -i` 

* Update your repository and your system 

```
sudo apt-get update && sudo apt-get upgrade -y

```

* Install haproxy

```
sudo apt-get install haproxy -y
```

* Edit haproxy configuration 

```
vi /etc/haproxy/haproxy.cfg
```

Add the below lines to create a frontend configuration for loadbalancer - 

```
frontend fe-apiserver
   bind 0.0.0.0:6443
   mode tcp
   option tcplog
   default_backend be-apiserver
```

Add the below lines to create a backend configuration for master1 and master2 nodes at port 6443. __**Note**__ : 6443 is the default port of **kube-apiserver**

```
backend be-apiserver
   mode tcp
   option tcplog
   option tcp-check
   balance roundrobin
   default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s maxconn 250 maxqueue 256 weight 100

       server master1 10.138.0.15:6443 check
       server master2 10.138.0.16:6443 check
```

Here - **master1** and **master2** are the names of the master nodes and **10.138.0.15** and **10.138.0.16** are the corresponding internal IP addresses. 

* Restart and Verify haproxy

```
systemctl restart haproxy
systemctl status haproxy
```

Ensure haproxy is in running status. 

**Note** If you see failures for master1 and master2 connectivity, you can ignore them for time being as we have not yet installed anything on the servers. 








