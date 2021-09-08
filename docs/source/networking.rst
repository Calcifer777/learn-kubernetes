#########################
Networking
#########################

Pre-requisites
****************

Switching and Routing
========================

Switching: a switch allows 2 or more devices to communicate in a local network
Routing: a router helps connect two or more networks

Useful commands: 

- `ip link`: list and modify interfaces on the host
- `ip addr`: see the IP addresses assigned to those interfaces
- `ip addr add <device-ip-address> dev eth0`: set IP addresses on the interfaces; these settings do not survive a host restart; to persist these changes, modify the `etc/network-interfaces` file
- `ip route`: view the routing table (used for cross network communications)
- `ip route add <device-ip-address> via <gateway-ip-address>`: add an entry in the routing table. To connect to the internet, for any new IP, use `ip route add default via <gateway-ip-address>`
- `cat /proc/sys/net/ipv4/ip_forward`: check if ip-forwarding is enabled on the host. To make it permanent, modyfy the `/etc/sysctl.conf` file

DNS
======

To assign a name to an IP address, add an entry to `/etc/hosts`.

To add a common DNS nameserver, add the entry `nameserver <ip-to-nameserver>` to the `/etc/resolv.conf` file.

By default, rules on the `hosts` file have priority on those in the nameserver; this behavior can be changed by modifying the `/etc/nsswitch.conf` (`docs <https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html>`_)

Record types:

- A: name to IPv4
- AAAA: name to IPv6
- CNAME: aliases, i.e. name to name

Useful commands:

- `search <alias> <hostname>`: configure an alias for a host
- `nslookup`: does not consider the local host file
- `dig`

Network Namespaces
===================

Useful commands:
- `ip netns`
- `ip netns add <network-namespace>`
- `ip netns exec <network-namespace> ip link`: executes a command within a network ns
- `ip link add <veth-2-name> type veth peer name <veth-2-name>`: create a pipe between two networks
- `ip link set veth-red netns red`: attach a veth to a network ns
- `ip -n <netns-name> addr add <some-ip-address> dev <veth-name>`: assign an IP address to the netns
- `ip -n <netns-name> link set <veth-name> up`: activate the veth

To allow multiple netns to communicate between each other on a single host, we create a virtual switch (via Linux Bridge, Open vSwitch, etc).

- `ip link add <vnet-name> type bridge`; create an internal bridge network
- `ip link set dev <vnet-name> up`; create an internal bridge network
- `ip link add <veth-name> type veth peer name <veth-bridge-name>`; create an internal bridge network

Steps:

1. Create network namespace 
2. Create Bridge network / interface 
3. Create VETH Pairs (pipe, virtual cable)
4. Attach VETH to namespace
5. Attach other VETH to Bridge
6. Assign IP addresses 
7. Bring the interfaces up
8. Enable NAT - IP masquerade

Docker networking
===================

Types of docker networking connectivity:

- `none`: the container is not attached to the `none` network; it cannot reach the outside world, and cannot communicate to the other contaienrs
- `host`: the container is attached to the host network - no netns is involved and there is no network separation between host and container
- `bridge`: create an internal private network which the docker host and container attach to

Cluster networking interface (CNI)
=======================================

The CNI is a set of standards that define how programs should be developed to solve networking challenges in a container runtime environment.

The programs are called *network plugins* (NP).

CNI defines a set of responsibilities for a Container Runtime and the NP:
- Container Runtime (RKT, K8s)
  - Create a netns 
  - Identify the network the container should attach to 
  - Invoke the NP when the container is added
  - Invoke the NP when the container is deleted
  - JSON format of the network configuration
- Network Plugin 
  - Support command line arguments (e.g. ADD, DEL, CHECK)
  - Support parameters (e.g. container id, netns)
  - Manage IP addresses assignment to PODs
  - Return results in a specific format

Docker does not implement a CNI

Cluster Networking
*********************

Port configuration:
- Master:
  - ETCD: 2379 (controplane components), 2380 (p2p connectivity)
  - Kubelet: 10250
  - Kubeapi: 6443
  - Kubelet: 10250
  - Kube-scheduler: 10251
  - Kube-controller-manager: 10252
- Workers
  - Kubelet: 10250

Pod networking
*********************

K8s expects that each node has a networking solution that makes it so that:

- every Pod has an IP address
- every Pod can communicate with every other Por in the same node
- every Pod can communicate with every other Pod in other nodes without a NAT

IPAM
*******

Service networking
*********************

Ingress
*********************

An ingress-controller is a reverse proxy that is exposed to external clients and routes the requests based on domain name, path, and protocol.

An ingress rules defines the routing behavior that is enforced by the ingress controller.

An ingress controller is composed by:
- a deployment of the reverse proxy
- a configmap to pass configuration settings to the reverse proxy
- a service exposing the deployment
- a service account, role, and role bindings to allow the ingress controller to monitor the cluster and align its configuration to the configured ingress rules

Requests not matching any of the ingress rules configured for the controller are routed towards a service named `default-http-backend`.

.. code-block:: yaml

  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-ingress
    annotations:
      nginx.ingress.kubernetes.io/rewrite-target: /  # this redirects requests destined to /path-1 to the root path
      nginx.ingress.kubernetes.io/ssl-redirect: "false"
  spec:
    rules:
    - host: this.is.my.domain.com
      http:
        paths:
        - path: /path-1
          pathType: Prefix
          backend:
            service:
              name: service-1
              port:
                number: 80
    - host: this.is.my.domain.com
      ...

Exercises
************

- Get IP address of node: `kubectl get nodes -o wide`
- Get CNI of node: `ip a | grep -B2 <node-ip>`
- Get mac-address of CNI of node: `ip a | grep -B2 <node-ip> | grep link/ether`
- Get MAC address of worker node `arp <node-name>`
- Get name of bridge interface created by Docker: `ip a | grep docker -A 1`
- Show default IP route: `ip route`
- identify the network plugin configured for Kubernetes: `ps aux | grep kubelet | grep network-plugin`
- Identify the CNI plugin used by K8s: `cd etc/cni/net.d/`