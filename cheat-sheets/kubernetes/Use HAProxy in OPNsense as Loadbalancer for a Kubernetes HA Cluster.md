# How to use the integrated HAProxy Package in OPNsense as LB for Kubernetes

When you are using a OPNsense in your Network e.g. in your Homelab as your central Firewall, you can use the integrated HAProxy Package to create a LB for a Kubernetes Cluster. So the usage of MetalLB etc. ist not necessary anymore

## Prerequisites
* Running OPNsense  [OPNsense Homepage](https://opnsense.org/)
* Running Kubernetes Cluster e.g. K3s

## Steps
### 0. Add a Virtual IP (optional)
If you want that the LB has a different IP than the OPNsense Node, then you should add a Virtual IP for the LB

Go to **Interfaces > Virtual IPs > Settings** and create an IP Alias:

* **Mode**: IP Alias
* **Interface**: Assign the Virtual IP to an Interface
* **Type**: Single address
* **Address**: <Any IP Address> /32 

### 1. Install HAProxy Plugin
 Go to **System > Firmware > Plugins** and search for '**os-haproxy**'  and press the '+'- Button

### 2. Configure HAProxy
#### 2.1 Add Real Servers
Go to **Services > HAProxy > Settings > Real Servers** and create an entry for every Kubernetes Node in the Cluster:

* **Type**: static
* ***FQDN or IP**: IP Address of the Kubernetes Node
* **Mode**: active

#### 2.2. Add a Backend Pool
Go to **Services > HAProxy > Settings > Virtual Services > Backend Pools** and create one Pool with a created Real Servers included:

* **Mode**: TCP (Layer 4)
* **Balancing Algorithm**: Round Robin
* **Servers**: Add every Real Server by Name
* **Enable Health Checking**: true

#### 2.3. Add a Public Service
Go to **Servcies > HAProxy > Settings > Virtual Services > Public Services** and create only **ONE** Service.

* **Listen Addresses**: <Virtual IP (LB)>:80 <Virtual IP (LB)>:443 <Virtual IP (LB)>:6443
* **Type**: SSL/HTTPS (TCP mode)
* **Default Backend Pool**: The name of the created Backend Pool

### 3. Configure Firewall Rules
#### 3.1 Alllow Access between the Kubernetes Nodes
Go to **Firewall > Rules > (Interface with assigned Virtual IP) and create the following Rule:

* **Protocol**: Any
* **Source**: IP Addresses of every Kubernetes Node
* **Destination**: IP Addresses of every Kubernetes Node

With the Rule we can confirm that the Nodes can talk to each other in the Cluster

#### 3.2 Allow Access to Kubernetes Cluster from Internal Network
Go to **Firewall > Rules> (Interface from which you need access) and create the following Rules:

a. Allow Access for Administration (kubectl) 
* **Protocol**: TCP/UDP
*  **Source**: Net or Hosts
*  **Destination**: <Virtual IP(LB)>
*  **Port**: 6443

b. Allow Access over HTTP
* **Protocol**: TCP
*  **Source**: Net or Hosts
*  **Destination**: <Virtual IP(LB)>
*  **Port**: 80

c. Allow Access over HTTPS
* **Protocol**: TCP
*  **Source**: Net or Hosts
*  **Destination**: <Virtual IP(LB)>
*  **Port**: 443