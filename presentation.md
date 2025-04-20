---
marp: true
theme: ./css/palette-3.css
paginate: true
class: font-xs
@auto-scaling: true
---
<!-- _paginate: skip -->
<!-- _footer: _Dale Bewley_ -->
<!-- class: bg-tertiary -->

# Deep dive into [CNI](https://cni.dev/)

---
<!-- backgroundColor: bg-secondary -->
<!-- class: bg-secondary -->

# Presentation Outline
## Intro & objectives

---
## CNI spec & OVN‑Kubernetes overview

### 2. OVN–Kubernetes (Primary CNI)

* **ovn‑kubernetes GitHub**  
  * Website: [ovn-kubernetes.io/](https://ovn-kubernetes.io/)
  * Repo: [ovn-org/ovn-kubernetes](https://github.com/ovn-org/ovn-kubernetes)  
  * Core repo for the OVN–Kubernetes integration: CNI binaries, controllers, docs.  
* **OpenShift OVN‑Kubernetes Guide**  
  * Docs: [OpenShift Container Platform Networking – OVN‑Kubernetes](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/networking/index)
  * Red Hat’s overview of

---
## Multus architecture

---

## Bridge plugin deep dive (config + demo)

---
## MAC VLAN plugin deep dive

---
## Debugging tools & workflows

---
## KubeVirt Interfaces and Networks

* Masquerade on Cluster Network
* L2 Bridge on Primary User Defined Network
* 
---
<!-- class: invert -->
<!-- header: Cluster Network -->
### VM Examples - Cluster Network

By default all VMs have the same address internally but masquerade using the pod IP. NAT happens on egress and selectively on ingress using Kubernetes Services.

```yaml
# VM attached to default cluster network
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-on-cluster-network
spec:
  template:
    spec:
      domain:
        devices:
          interfaces:
            - macAddress: '02:86:5e:00:00:07'
              masquerade: {}
              model: virtio
              name: default
      networks:
        - name: default
          pod: {}        
```

---
<!-- class: default -->

Two ethernet interfaces in the pod. One on infrastructure locked cluster network & one on 10.0.2.1/24 in every virt launcher pod.

```bash
sh-5.1$ ip -c link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0@if379: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 0a:58:0a:83:01:61 brd ff:ff:ff:ff:ff:ff link-netnsid 0
3: k6t-eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 02:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
4: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel master k6t-eth0 state UP mode DEFAULT group default qlen 1000
    link/ether be:53:ae:c8:c5:66 brd ff:ff:ff:ff:ff:ff

sh-5.1$ ip -br -c -4 a
lo               UNKNOWN        127.0.0.1/8 
eth0@if379       UP             10.131.1.97/23 
k6t-eth0         UP             10.0.2.1/24  
```

Ethernet interface in the VM _always_ has IP `10.0.2.2/24`. Masquerades as pod IP `10.131.1.97/23` above.

```bash
[cloud-user@vm-pod ~]$ ip -c link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 02:86:5e:00:00:10 brd ff:ff:ff:ff:ff:ff
    altname enp1s0

[cloud-user@vm-pod ~]$ ip -br -c -4 a
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             10.0.2.2/24 

[cloud-user@vm-pod ~]$ ip -c route
default via 10.0.2.1 dev eth0 proto dhcp src 10.0.2.2 metric 100 
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.2 metric 100 
```

---
# VM on Default Cluster Network 

## Interfaces

### Virt-launcher Pods
* eth0 on cluster network `10.128.0.0/14`
* k6t-eth0 always has IP `10.0.2.1/24`

### VirtualMachines
* eth0 always has IP `10.0.2.2/24`
* Default gateway is `10.0.2.1` on virt-launcher pod
* Masquerades at pod edge as IP of the virt-launcher pod
* Masquerades at node edge as IP of node default interface `br-ex`

---
<!-- _class: invert -->
<!-- header: Primary User Defined Network -->
### VM Examples - Primary UDN
In this example all VMs have the same address internally and masquerade using the pod IP on egress.
```yaml
# VM attached to primary UDN
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-on-primary-udn
spec:
  template:
    spec:
      domain:
        devices:
          interfaces:
            - binding:
                name: l2bridge
              model: virtio
              name: default      
      networks:
        - name: default
          pod: {}        
```

---
<!-- class: default -->

Two ethernet interfaces in the pod. One on infrastructure locked cluster network & one on UDN.

```bash
sh-5.1$ ip -c link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0@if356: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default 
    link/ether 0a:58:0a:83:01:4b brd ff:ff:ff:ff:ff:ff link-netnsid 0
3: ovn-udn1-nic@if357: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue master k6t-ovn-udn1 state UP mode DEFAULT group default 
    link/ether 06:1b:c3:df:4d:d3 brd ff:ff:ff:ff:ff:ff link-netnsid 0
4: k6t-ovn-udn1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 06:1b:c3:df:4d:d3 brd ff:ff:ff:ff:ff:ff
5: tap0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel master k6t-ovn-udn1 state UP mode DEFAULT group default qlen 1000
    link/ether 82:38:45:e8:1a:3d brd ff:ff:ff:ff:ff:ff
6: ovn-udn1: <BROADCAST,NOARP> mtu 1400 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 0a:58:0a:01:01:03 brd ff:ff:ff:ff:ff:ff

sh-5.1$ ip -br -c -4 a
lo               UNKNOWN        127.0.0.1/8 
eth0@if356       UP             10.131.1.75/23 
ovn-udn1         DOWN           10.1.1.3/24 
```

One ethernet interface in the VM with IP from primary UDN

```bash
[cloud-user@vm-primary-udn ~]$ ip -c link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
    link/ether 0a:58:0a:01:01:03 brd ff:ff:ff:ff:ff:ff
    altname enp1s0

[cloud-user@vm-primary-udn ~]$ ip -br -c -4 a
lo               UNKNOWN        127.0.0.1/8 
eth0             UP             10.1.1.3/24 

[cloud-user@vm-primary-udn ~]$ ip -c route
default via 10.1.1.1 dev eth0 proto dhcp src 10.1.1.3 metric 100 
10.1.1.0/24 dev eth0 proto kernel scope link src 10.1.1.3 metric 100 
```
---
# VM on Primary User Defined Network 

## Interfaces

### Virt-launcher Pods
* Two ethernet interfaces
* eth0@if356 is on infrastructure locked cluster network `10.128.0.0/14`
* ovn-udn1 is on primary UDN `10.1.1.3/24`

### VirtualMachines
* eth0 has IP `10.1.1.3/24` from primary UDN
* Default gateway is `10.1.1.1`
* Masquerades at UDN gateway rotuer as the IP from `169.254.0.0/17` associated with the UDN
* Masquerades at node edge as IP of node default interface `br-ex`
---

## Masquerade Subnet
Each UDN has two IPs allocated from the masquerade subnet.
```yaml
# oc get network.operator/cluster -o yaml
apiVersion: operator.openshift.io/v1
kind: Network
metadata:
  name: cluster
spec:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  defaultNetwork:
    ovnKubernetesConfig:
      egressIPConfig: {}
      gatewayConfig:
        ipv4: {} # <-- default: 169.254.0.0/17
        ipv6: {} # <-- default: fd69::/125
        routingViaHost: false
```

---
<!-- _class: invert -->
<!-- header: Primary & Secondary User Defined Networks -->
### VM Examples - Primary and Secondary UDN

```yaml
# VM attached to primary UDN and secondary UDN
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-on-primary-udn
spec:
  template:
    spec:
      domain:
        devices:
          interfaces:
            - binding:
                name: l2bridge
              model: virtio
              name: default
            - bridge: {}
              macAddress: '02:86:5e:00:00:0a'
              model: virtio
              name: nic-secondary-udn
      networks:
        - name: default
          pod: {}        
        - multus:
            networkName: secondary-udn
          name: nic-secondary-udn
```

---
<!-- class: default -->

* Multus log:


---

* Pod annotations

```json
oc get pods -n demo-udn-2 virt-launcher-fedora-black-elephant-21-msldg -o json \
  | jq -r  '.metadata.annotations."k8s.ovn.org/pod-networks"' | jq -s
...
```

---
## Q&A / further reading

* [CNI Spec](https://github.com/containernetworking/cni/blob/main/SPEC.md)
