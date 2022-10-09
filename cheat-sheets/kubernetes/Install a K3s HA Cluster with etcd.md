# Install K3S in High Availability Mode with an embedded Database
## Install first Server
```bash
curl -sfL https://get.k3s.io | sh -s - server \
--token=YOUR-SECRET \
--tls-san your-dns-name --tls-san your-lb-ip-address \
--cluster-init
```

### SSL Certificates
To avoid certificate errors in such a configuration, you should install the server with the `--tls-san YOUR_IP_OR_HOSTNAME_HERE` option. This option adds an additional hostname or IP as a Subject Alternative Name in the TLS cert, and it can be specified multiple times if you would like to access via both the IP and the hostname.

## Install additional Servers
```bash
curl -sfL https://get.k3s.io | sh -s - server \
--token=YOUR-SECRET \
--tls-san your-dns-name --tls-san your-lb-ip-address \
--server https://IP-OF-THE-FIRST-SERVER:6443
```

## Install Agents
You can still add additional nodes without a server function to this cluster.
```bash
curl -sfL https://get.k3s.io | sh -s - agent \
--server https://your-lb-ip-address:6443 \
--token YOUR-SECRET
``` 

## ETCD Fault Tolerance
The `--cluster-init` initializes an HA Cluster with an embedded etcd database. The fault tolerance requires an odd number, minimum three, nodes to function.

Total Number of nodes | Failed Node Tolerance
---|---
1|0
2|0
3|1
4|1
5|2
6|2
...|...

