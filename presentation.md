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

```bash
cat foo.sh
```

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

One ethernet interface in the pod

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

Ethernet interface in the VM _always_ has IP `10.0.2.2/24`. Masquerades as pod IP (`10.131.1.97/23` above).

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
```

---
<!-- _class: invert -->
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
```

---
<!-- _class: invert -->
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

```
2025-04-19T19:19:58Z [verbose] ADD starting CNI request ContainerID:"663af540f32650814f06579e37f2a59505d8285ff4beb60f2ae905a5cbf4ec30" Netns:"/var/run/netns/42c19e0e-af60-4d5b-8714-9c446ab8ac83" IfName:"eth0" Args:"IgnoreUnknown=1;K8S_POD_NAMESPACE=demo-udn-2;K8S_POD_NAME=virt-launcher-fedora-black-elephant-21-msldg;K8S_POD_INFRA_CONTAINER_ID=663
af540f32650814f06579e37f2a59505d8285ff4beb60f2ae905a5cbf4ec30;K8S_POD_UID=526e5f4e-f646-4aec-b3f6-1f9ccc33f330" Path:""
2025-04-19T19:19:59Z [verbose] Add: demo-udn-2:virt-launcher-fedora-black-elephant-21-msldg:526e5f4e-f646-4aec-b3f6-1f9ccc33f330:ovn-kubernetes(ovn-kubernetes):eth0 {"cniVersion":"0.4.0","interfaces":[{"name":"663af540f326508","mac":"f6:f5:bc:30:ee:d3"},{"name":"eth0","mac":"0a:58:0a:80:00:ba","sandbox":"/var/run/netns/42c19e0e-af60-4d5b-8714-9c4
46ab8ac83"},{"name":"663af540f3265_3","mac":"8e:83:e5:4a:83:a9"},{"name":"ovn-udn1","mac":"0a:58:c0:a8:de:03","sandbox":"/var/run/netns/42c19e0e-af60-4d5b-8714-9c446ab8ac83"}],"ips":[{"version":"4","interface":1,"address":"10.128.0.186/23"},{"version":"4","interface":3,"address":"192.168.222.3/24","gateway":"192.168.222.1"}],"dns":{}}
I0419 19:19:59.545551    7342 event.go:364] Event(v1.ObjectReference{Kind:"Pod", Namespace:"demo-udn-2", Name:"virt-launcher-fedora-black-elephant-21-msldg", UID:"526e5f4e-f646-4aec-b3f6-1f9ccc33f330", APIVersion:"v1", ResourceVersion:"48371528", FieldPath:""}): type: 'Normal' reason: 'AddedInterface' Add eth0 [10.128.0.186/23 192.168.222.3/24] f
rom ovn-kubernetes
2025-04-19T19:20:00Z [verbose] Add: demo-udn-2:virt-launcher-fedora-black-elephant-21-msldg:526e5f4e-f646-4aec-b3f6-1f9ccc33f330:demo-udn-2/secondary-udn(demo-udn-2_secondary-udn):podc00fa7e7651 {"cniVersion":"1.0.0","interfaces":[{"mac":"8a:16:eb:9a:28:6b","name":"663af540f3265_4"},{"mac":"02:86:5e:00:00:0a","name":"podc00fa7e7651","sandbox":"/v
ar/run/netns/42c19e0e-af60-4d5b-8714-9c446ab8ac83"}],"ips":[{"address":"10.2.2.1/24","interface":1}]}                                                                                                                                                                                                                                                       I0419 19:20:00.352145    7342 event.go:364] Event(v1.ObjectReference{Kind:"Pod", Namespace:"demo-udn-2", Name:"virt-launcher-fedora-black-elephant-21-msldg", UID:"526e5f4e-f646-4aec-b3f6-1f9ccc33f330", APIVersion:"v1", ResourceVersion:"48371528", FieldPath:""}): type: 'Normal' reason: 'AddedInterface' Add podc00fa7e7651 [10.2.2.1/24] from demo-udn-2/secondary-udn
```

---

* Pod annotations

```json
oc get pods -n demo-udn-2 virt-launcher-fedora-black-elephant-21-msldg -o json | jq -r  '.metadata.annotations."k8s.ovn.org/pod-networks"' | jq -s
[
  {
    "default": {
      "ip_addresses": [
        "10.128.0.186/23"
      ],
      "mac_address": "0a:58:0a:80:00:ba",
      "routes": [
        {
          "dest": "10.128.0.0/14",
          "nextHop": "10.128.0.1"
        },
        {
          "dest": "100.64.0.0/16",
          "nextHop": "10.128.0.1"
        }
      ],
      "ip_address": "10.128.0.186/23",
      "role": "infrastructure-locked"
    },
    "demo-udn-2/primary-udn": {
      "ip_addresses": [
        "192.168.222.3/24"
      ],
      "mac_address": "0a:58:c0:a8:de:03",
      "gateway_ips": [
        "192.168.222.1"
      ],
      "routes": [
        {
          "dest": "172.30.0.0/16",
          "nextHop": "192.168.222.1"
        },
        {
          "dest": "100.65.0.0/16",
          "nextHop": "192.168.222.1"
        }
      ],
      "ip_address": "192.168.222.3/24",
      "gateway_ip": "192.168.222.1",
      "tunnel_id": 9,
      "role": "primary"
    },
    "demo-udn-2/secondary-udn": {
      "ip_addresses": [
        "10.2.2.1/24"
      ],
      "mac_address": "02:86:5e:00:00:0a",
      "ip_address": "10.2.2.1/24",
      "tunnel_id": 4,
      "role": "secondary"
    }
  }
]
```

---
## Q&A / further reading

* [CNI Spec](https://github.com/containernetworking/cni/blob/main/SPEC.md)
