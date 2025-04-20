## CNI Specification

- **Container Network Interface (CNI) Spec**  
  - GitHub: [containernetworking/cni SPEC.md](https://github.com/containernetworking/cni/blob/main/SPEC.md)  
  - Defines the CNI plugin interface, JSON config schema, versioning, and error handling.

## OVN–Kubernetes (Primary CNI)

- **ovn‑kubernetes GitHub**  
  - Website: [ovn-kubernetes.io/](https://ovn-kubernetes.io/)
  - Architecture: [ovn-kubeernetes/design/architecture](https://ovn-kubernetes.io/design/architecture/)
  - Repo: [ovn-org/ovn-kubernetes](https://github.com/ovn-org/ovn-kubernetes)  
  - Core repo for the OVN–Kubernetes integration: CNI binaries, controllers, docs.  
- **OpenShift OVN‑Kubernetes Guide**  
  - Docs: [OpenShift Container Platform Networking – OVN‑Kubernetes](https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/networking/index)
  - Red Hat’s overview of OVN‑powered networking on OpenShift.

## Multus CNI (Meta‑Plugin)

- **multus‑cni GitHub**  
  - Repo: [k8snetworkplumbingwg/multus-cni](https://github.com/k8snetworkplumbingwg/multus-cni)  
  - Implements the NetworkAttachmentDefinition CRD and CNI chaining logic.  
- **Network Plumbing WG Multus Spec**  
  - Docs: [multus-cni/docs/multus.md](https://github.com/k8snetworkplumbingwg/multus-cni/blob/master/docs/multus.md)  
  - Detailed design doc, config examples, and recommendations.

## Bridge Plugin

- **bridge CNI Plugin (containernetworking/plugins)**  
  - Repo: [containernetworking/plugins/tree/main/plugins/main/bridge](https://github.com/containernetworking/plugins/tree/main/plugins/main/bridge)  
  - Source: [bridge.go](https://github.com/containernetworking/plugins/blob/main/plugins/main/bridge/bridge.go)

- **Bridge Plugin README/Examples**  
  - Docs: [bridge/README.md](https://cni.dev/plugins/current/main/bridge/)
  - Usage examples with host-local IPAM, DHCP, hairpin mode, etc.

## MAC VLAN Plugin

- **macvlan CNI Plugin (containernetworking/plugins)**  
  - Repo: [containernetworking/plugins/tree/main/plugins/main/macvlan](https://github.com/containernetworking/plugins/tree/main/plugins/main/macvlan)  
  - Source: [macvlan.go](https://github.com/containernetworking/plugins/blob/main/plugins/main/macvlan/macvlan.go)

- **MACVLAN Plugin README**  
  - Docs: [macvlan/README.md](https://www.cni.dev/plugins/current/main/macvlan/)
  - Modes (bridge, private, passthru), host interface requirements.

## OpenShift Network Operator

- **Network Operator Docs**  
  - Docs: [Configuring cluster network (Network CR)](https://docs.openshift.com/container-platform/latest/networking/configuring_cluster_network/configuring-cluster-network.html)  
  - How OVN is configured and tuned via the `Network` CR.

## KubeVirt Networking

- **KubeVirt GitHub**  
  - Repo: [kubevirt/kubevirt](https://github.com/kubevirt/kubevirt)  
  - Core VMI definitions, virt-launcher pods, network plumbing.  
- **KubeVirt User Guide: Networking**  
  - Docs: [kubevirt.io/user-guide/network](https://kubevirt.io/user-guide/network/)  
  - Types of networks (pod, multus, SR‑IOV), YAML examples, how tap devices are wired.

### KubeVirt Interfaces and Networks

- **VM Interfaces**
  - Docs: [kubervirt.io/interfaces_and_networks](https://kubevirt.io/user-guide/network/interfaces_and_networks/)

  - networks are specified in spec.networks. Then, interfaces backed by the networks are added to the VM by specifying them in spec.domain.devices.interfaces.
### KueVirt Supported Plugins

- **Network Binding Plugins**
  - Docs: [kubevirt.io/network_binding_plugins](https://kubevirt.io/user-guide/network/network_binding_plugins/)

- **Passt Binding**
  - Docs: [net_binding_plugins/passt](https://kubevirt.io/user-guide/network/net_binding_plugins/passt/)
  - User space translation between layer-2 interface and layer-4 sockets on a host

## 8. Debugging & Inspection Tools

- **OVN CLI Reference**  
  - Docs: [ovn/README.md](https://opendev.org/ovn/ovn/src/branch/master/README.md#ovn-nbctl-and-ovn-sbctl)  
- **ovn-trace Docs**  
  - Docs: [ovn-trace/README.md](https://opendev.org/ovn/ovn/src/branch/master/ovn-trace/README.md)  
  - How to simulate and trace logical flows.  
- **Kubectl & OpenShift Network Debug**  
  - Commands: `kubectl debug`, `oc adm inspect network.operator` in the official kubectl/oc man pages.
