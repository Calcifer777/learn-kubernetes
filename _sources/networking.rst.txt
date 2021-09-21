#########################
Networking
#########################

****************
Pre-requisites
****************

Switching and Routing
========================

- Switch / Network Interface: allows 2 or more devices to communicate in a local network
- Router: connects two or more networks

Useful commands: 

- `ip link`: list and modify interfaces on the host
- `ip addr`: see the IP addresses assigned to those interfaces
- `ip addr add <cidr-block> dev <dev(ice)-name>`: configure IP address routing via an interface. These settings do not survive a host restart; to persist these changes, modify the `/etc/network/interfaces` file
- `ip route`: view the routing table (used for cross network communications)
- `ip route add <cidr-block> via <gateway-ip-address>`: add an entry in the routing table. To connect to the internet, for any new IP, use `ip route add default via <gateway-ip-address>`
- `cat /proc/sys/net/ipv4/ip_forward`: check if ip-forwarding is enabled on the host. To make it permanent, modify the `/etc/sysctl.conf` file

DNS
========================

To assign a name to an IP address, add an entry to `/etc/hosts`.

To add a common DNS nameserver, add the entry `nameserver <ip-to-nameserver>` to the `/etc/resolv.conf` file.

By default, rules on the `hosts` file have priority on those in the nameserver; this behavior can be changed by modifying the `/etc/nsswitch.conf` (`docs <https://man7.org/linux/man-pages/man5/nsswitch.conf.5.html>`_)

Entry types that can be found in the `/etc/hosts` file:
- A record: maps a name to IPv4
- AAAA record: maps a name to IPv6
- CNAME record: aliases, i.e. maps a name to name
- `search <alias> <hostname>`: configure an alias for a host
- `search <domain>`: configures a domain name under which to search unresolved domains

Useful commands:

- `nslookup`: does not consider the local host file
- `dig`

Network Namespaces
========================

ARP TABLE (Address Resolution Table) Resolves IP address to Mac Address's. 

https://man7.org/linux/man-pages/man4/veth.4.html

All modern operating systems come equipped with a firewall – a software application that regulates network traffic to a computer. Firewalls create a barrier between a trusted network (like an office network) and an untrusted one (like the internet). Firewalls work by defining rules that govern which traffic is allowed, and which is blocked. The utility firewall developed for Linux systems is iptables.

To allow multiple netns to communicate between each other on a single host, we create a virtual switch (via Linux Bridge, Open vSwitch, etc).

Docker networking
===================

Types of docker networking connectivity:

- `none`: the container is not attached to the `none` network; it cannot reach the outside world, and cannot communicate to the other contaienrs
- `host`: the container is attached to the host network - no `netns` is involved and there is no network separation between host and container
- `bridge`: create an internal private network which the docker host and container attach to

************************************
Cluster networking interface (CNI)
************************************

The CNI is a set of standards that define how programs should be developed to solve networking challenges in a container runtime environment.

The programs that implement a CNI are called *(network) plugins* (NP).

CNI defines a set of responsibilities for a Container Runtime and the NP:
- Container Runtime (RKT, K8s)
  - Create a netns 
  - Identify the network the container should attach to 
  - Invoke the NP when the container is added
  - Invoke the NP when the container is deleted
  - JSON format of the network configuration
- Plugin (bridge, weave, Flannel, Calico)
  - Support command line arguments (e.g. ADD, DEL, CHECK)
  - Support parameters (e.g. container id, netns)
  - Manage IP addresses assignment to PODs
  - Return results in a specific format

Docker does not implement a CNI; Docker implements CNM (container network model). K8s implements networking by creating container in the `none` network and then invoking the CNI script to allow connectivity.

************************************
Cluster Networking
************************************

Port configuration:
- Master:
  - Kubeapi: 6443
  - Kubelet: 10250
  - Kube-scheduler: 10251
  - Kube-controller-manager: 10252
  - ETCD: 2379 (controplane components), 2380 (p2p connectivity)
- Workers
  - Kubelet: 10250

************************************
Pod networking
************************************

K8s expects that each node has a networking solution that makes it so that:

- every Pod has an IP address
- every Pod can communicate with every other Pod in the same node
- every Pod can communicate with every other Pod in other nodes without a NAT

The CNI plugin executes the necessary commands to ensure inter-pod communication in the cluster.

The kubelet uses the CNI configuration passed as cli-argument o in the pod specification, at the entries:

- `network-plugin: cni`
- `cni-bin-dir`: where all the CNI scripts are placed. Defaults to `/opt/cni/bin/`
- `cni-conf-dir`: where the CNI configuration is placed; points to one of the scripts in the bin directory. If multiple files are present, it chooses the first one in alphabetical order

Example
------------------------------------

.. code-block:: json

  {
    "cniVersion": "0.2.0",
    "name": "my-net",
    "type": "bridge",
    "bridge": "cni0",
    "isGateway": true,
    "isMasq": true,
    "ipam": {   # 
      "type": "host-local",
      "subnet": "10.22.0.0/16",  # range of IP addresses assigned to Pods
      "routes": [
        "dst": "0.0.0.0/0"
      ]
    }
  }


************************************
IP Address Management (IPAM)
************************************

IPAM comprehends
- How the virtual bridge networks in the nodes are assigned an IP subnet
- How the Pods are assigned an IP

The CNI plugins manages IPAM tasks.

CNI comes with 2 built in plugins to manage IPAM:
- DHCP 
- Host-local

************************************
Service networking
************************************

`Kube-proxy` is responsible for exposing the services IP to the Pods in the cluster and external clients.

Services are cross-nodes K8s objects.

When a service is created, `kube-proxy`:

- assigns an internal IP to that service. The IP is choosen randomly from a list of free IPs in a given range. The range is configured wia the `service-cluster-ip-range` cli argument; this range should not overlap with the one for IP assigned to the Pods (set in the `kube-apiserver`)
- adds forwarding rules in each Node that map the IP and port of the service to the IP and port of the Pod.

`kube-proxy` can enforce these forwarding rules via:

- iptables
- userspace
- ipvs


************************************
DNS in K8s
************************************

The default DNS server in K8s is CoreDNS. It is deployed as a `deployment` with replicas for redundancy.

CoreDNS is configured via a `Corefile`, for example:

.. code-block:: json

  {
    errors
    health
    kubernetes cluster.local in addr.arpa ip6.arpa {
      pods insecure
      upstreams
      fallthrough in-addr.arpa ip6.arpa
    }
    prometheus :9153
    proxy . /etc/resolv.conf
    cache 30
    reload
  }

The fully qualified domain name for a service is:
`<service-name>.<namespace>.svc.cluster.local`

Pods can have DNS records, but the fist component of the FQDN is just the internal IP of the Pod, with periods replaced by dashes; that is `<pod-ip-with-dashes.default.pod>`

************************************
Ingress
************************************

An ingress-controller is a reverse proxy that is exposed to external clients and routes the requests based on domain name, path, and protocol.

An ingress rules defines the routing behavior that is enforced by the ingress controller.

An ingress controller is composed by:
- a `deployment` of the reverse proxy
- a `configmap` to pass configuration settings to the reverse proxy
- a `service` exposing the deployment
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


************************************
Exercises
************************************

Create and pair two network namespaces
================================================================

.. code-block:: bash

  ip netns add red
  ip netns add blue
  ip netns exec red ip link
  ip netns exec blue ip link
  ip link add veth-red type veth peer name veth-blue
  ip link set veth-red netns red
  ip link set veth-blue netns blue
  ip -n red addr add 192.168.15.1/24 dev veth-red
  ip -n blue addr add 192.168.15.2/24 dev veth-blue
  ip -n red link set veth-red up
  ip -n blue link set veth-blue up
  # Test that it is reachable from ns and from host
  # sudo ip netns exec blue ping 192.168.15.1


Create a network bridge that connects two network namespaces
================================================================

.. code-block:: bash

  # Create ns:
  ip netns add red
  ip netns add blue
  # Create interface bridge:
  ip link add v-net-0 type bridge
  # Change bridge status:
  ip link set dev v-net-0 up
  # Create vritual cables:
  ip link add veth-red type veth peer name veth-red-br
  ip link add veth-blue type veth peer name veth-blue-br
  # Join peer with ns:
  ip link set veth-red netns red
  ip link set veth-blue netns blue
  ip link set veth-red-br master v-net-0
  ip link set veth-blue-br master v-net-0
  # Add IP addresses to ns:
  ip -n red addr add 192.168.15.1/24 dev veth-red
  ip -n blue addr add 192.168.15.2/24 dev veth-blue
  # Bring up ns interfaces:
  ip -n red link set veth-red up
  ip -n blue link set veth-blue up
  # Add IP address to host:
  ip addr add 192.168.15.5/24 dev v-net-0
  # Bring up all interfaces on host:
  ip link set dev veth-red-br up
  ip link set dev veth-blue-br up
  # Test that it is reachable from ns and from host
  # sudo ip netns exec blue ping 192.168.15.1


Get node IP, network interface name, MAC
================================================================

.. code-block:: bash

  # Get node IP
  kubectl get nodes <nodename> -o wide  # option 1
  kubectl get nodes <nodename> -o jsonpath='{.status.addresses[?(@.type == "InternalIP")].address}'  # option 2
  # Get all interfaces with their IP addresses and filter by node IP
  ip addr | grep <node-ip> -B2
  # Get MAC address of worker node
  arp <node-name>


Get CNI configuration
================================================================

.. code-block:: bash
  # CNI configuration details are provided to the kubelet as cli arguments. 
  # If not present in the kubelet process, given by:
  ps aux | grep kubelet | grep cni

  # Get the default values from the help of the kubelet command
  kubelet -i | grep cni


Other
================================================================

- Get IP address of node: `kubectl get nodes -o wide`
- Get CNI of node: `ip a | grep -B2 <node-ip>`
- Get MAC-address of CNI of node: `ip a | grep -B2 <node-ip> | grep link/ether`
- Get MAC address of worker node `arp <node-name>`
- Get name of bridge interface created by Docker: `ip a | grep docker -A 1`
- Show default IP route: `ip route`
- identify the network plugin configured for Kubernetes: `ps aux | grep kubelet | grep network-plugin`
- Identify the CNI plugin used by K8s: `cd etc/cni/net.d/`
