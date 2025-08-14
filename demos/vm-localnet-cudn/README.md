# UDN testing

Here is a Localnet CUDN:

* [clusteruserdefinednetwork.yaml](../../components/localnet-udn/clusteruserdefinednetwork.yaml)
```yaml
---
# https://ovn-kubernetes.io/api-reference/userdefinednetwork-api-spec/#clusteruserdefinednetworkspec
apiVersion: k8s.ovn.org/v1
kind: ClusterUserDefinedNetwork
metadata:
  name: cudn-1924
spec:
  # what namespaces should have access to this network
  namespaceSelector:
    matchLabels:
      allow-cudn-1924: "true"
    matchExpressions:
      - key: kubernetes.io/metadata.name
        operator: In
        values: ["demo-vm-localnet-cudn"]
  network:
    topology: Localnet
    localnet:
      role: Secondary
      physicalNetworkName: "physnet-br-vmdata"
      vlan:
        mode: Access
        access:
          id: 1924
      subnets:
        - "192.168.4.0/24"
      # do not dole out these IPs
      excludeSubnets:
        - "192.168.4.0/25"
        - "192.168.4.128/26"
        - "192.168.4.240/28"
```

I started a VM with the above localnet CUDN on the 2nd NIC.

I see that the CNI selected the IP 192.168.4.194 for this pod interface. You can see that in this annotation.

```json
$ Oc get pods -n demo-vm-localnet-cudn virt-launcher-rhel9-1924-prdr4 -o yaml | yq '.metadata.annotations."k8s.ovn.org/pod-networks"' | jq
{
  "default": {
    "ip_addresses": [
      "10.129.4.35/23"
    ],
    "mac_address": "0a:58:0a:81:04:23",
    "gateway_ips": [
      "10.129.4.1"
    ],
    "routes": [
      {
        "dest": "10.128.0.0/14",
        "nextHop": "10.129.4.1"
      },
      {
        "dest": "172.30.0.0/16",
        "nextHop": "10.129.4.1"
      },
      {
        "dest": "169.254.0.5/32",
        "nextHop": "10.129.4.1"
      },
      {
        "dest": "100.64.0.0/16",
        "nextHop": "10.129.4.1"
      }
    ],
    "ip_address": "10.129.4.35/23",
    "gateway_ip": "10.129.4.1",
    "role": "primary"
  },
  "demo-vm-localnet-cudn/cudn-1924": {
    "ip_addresses": [
      "192.168.4.194/24"
    ],
    "mac_address": "02:a8:3f:00:00:08",
    "ip_address": "192.168.4.194/24",
    "role": "secondary"
  }
}
```
Once the VM booted, I saw this intiial discover which seemed to be a reused MAC address with an old lease which was discarded.

The client then asked for the IP which was assigned to it by the CNI. The dhcp server obliged.

```
[root@infra ~]# grep -i "02:a8:3f:00:00:08" /var/log/messages
Aug 13 14:15:55 infra dhcpd[1569]: DHCPDISCOVER from 02:a8:3f:00:00:08 via ens224
Aug 13 14:15:55 infra dhcpd[1569]: uid lease 192.168.4.80 for client 02:a8:3f:00:00:08 is duplicate on 192.168.4.0/24
Aug 13 14:15:55 infra dhcpd[1569]: DHCPREQUEST for 192.168.4.194 (169.254.75.11) from 02:a8:3f:00:00:08 via ens224
Aug 13 14:15:55 infra dhcpd[1569]: DHCPACK on 192.168.4.194 to 02:a8:3f:00:00:08 (rhel9-1924) via ens224
Aug 13 14:15:56 infra dhcpd[1569]: DHCPOFFER on 192.168.4.80 to 02:a8:3f:00:00:08 (rhel9-1924) via ens224
```

This DHCP request seems to be coming from NetworkManager in the VirtualMachine, I presume.


Can I set the default route via annotation? HOw?


# OVN Inspection

Let's look at the networking on the node where this VM launched.

```bash
oc get vmi/rhel9-1924 -n demo-vm-localnet-cudn -o wide
NAME         AGE   PHASE     IP            NODENAME              READY   LIVE-MIGRATABLE   PAUSED
rhel9-1924   77m   Running   10.129.4.35   hub-v57jl-cnv-vtlqc   True    True


ovncli hub-v57jl-cnv-vtlqc ovn-nbctl --bare --columns=name list Logical_Switch |sort -r
vlan.1924_ovn_localnet_switch
transit_switch
join
hub-v57jl-cnv-vtlqc
ext_hub-v57jl-cnv-vtlqc
ext_demo.vm.secondary.udn_primary.udn_hub-v57jl-cnv-vtlqc
ext_demo.vm.primary.udn_primary.udn_hub-v57jl-cnv-vtlqc
ext_demo.sandbox_primary.udn_hub-v57jl-cnv-vtlqc
demo.vm.secondary.udn_secondary.udn_ovn_layer2_switch
demo.vm.secondary.udn_primary.udn_ovn_layer2_switch
demo.vm.primary.udn_primary.udn_ovn_layer2_switch
demo.sandbox_tertiary.udn_ovn_layer2_switch
demo.sandbox_secondary.udn_ovn_layer2_switch
demo.sandbox_quaternary.udn_ovn_layer2_switch
demo.sandbox_primary.udn_ovn_layer2_switch
default.vlan.4_ovn_localnet_switch
cluster_udn_cudn.1924_ovn_localnet_switch
```