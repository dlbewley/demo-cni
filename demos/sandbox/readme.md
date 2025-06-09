# WIP

* [networks](base/kustomization.yaml) - ( [primary](../components/primary-udn/userdefinednetwork.yaml), [secondary](../components/secondary-udn/userdefinednetwork.yaml), [tertiary](../components/tertiary-udn/userdefinednetwork.yaml), [quaternary](../components/quaternary-udn/userdefinednetwork.yaml) )
* [firewall](vm-firewall/base/kustomization.yaml)
* [server-1](vm-server-1/base/kustomization.yaml)
* [server-2](vm-server-2/base/kustomization.yaml)
* [server-3](vm-server-3/base/kustomization.yaml)

> [!NOTE]
>  There can be only _ONE_ Primary User Defined Network for a namespace.

A _Primary_ Layer2 UDN will have a gateway-router created at IP `x.x.x.1` and pods will use this as their default route. This UDN will be assigned a masquerade IP and traffic will NAT as it egresses towards `br-ex` where it will NAT again as the node IP.

A _Secondary_ Layer2 UDN will have no gateway-router. The network is simply an overlay with no egress, unless a router is built by the user. ie. The Firewall here.

```mermaid
graph TD;
    subgraph nets-1["Primary UDN"]
        net-1st[primary-udn<br> 10.1.1.0/24]:::net-1st;
        net-1st --- net-1st-gr[gateway-router<br> 10.1.1.1/24]
    end

    subgraph nets-2["Secondary UDNs"]
        net-2nd[secondary-udn<br>10.2.2.0/24<br>]:::net-2nd;
        net-3rd[tertiary-udn<br>10.3.3.0/24<br>]:::net-3rd;
        net-4th[quaternary-udn<br>10.4.4.0/24<br>]:::net-4th;
    end

    subgraph Firewall
        eth0[eth0<br>10.1.1.9/24]:::interface;
        eth0 ==> net-1st
        eth1[eth1<br>10.2.2.2/24]:::interface;
        net-2nd --> eth1;
        eth2[eth2<br>10.3.3.1/24]:::interface;
        net-3rd --> eth2;
        eth3[eth3<br>10.4.4.1/24]:::interface;
        net-4th --> eth3;
    end

    subgraph Server-1
        server-1-eth0[eth0<br>10.1.1.10/24]:::interface;
        server-1-eth0 --> net-1st
        server-1-eth1[eth1<br>10.2.2.3/24]:::interface;
        server-1-eth1 --> net-2nd
    end

    subgraph Server-2
        server-2-eth0[eth0<br>10.2.2.1/24]:::interface;
        server-2-eth0 --> net-2nd
    end

    subgraph Server-3
        server-3-eth0[eth0<br>10.3.3.2/24]:::interface;
        server-3-eth0 --> net-3rd
        server-3-eth1[eth1<br>10.4.4.2/24]:::interface;
        server-3-eth1 --> net-4th
    end


    net-1st-gr --(Masq)--> br-ex --(Masq)--> internet

    classDef interface fill:#ffcc00,stroke:#333,stroke-width:2px;
    classDef network fill:#ddd,stroke:#333,stroke-width:2px;
    classDef neighbor fill:#ffeb99,stroke:#333,stroke-width:2px;
    classDef net-1st fill:#00ffff,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef net-2nd fill:#00dddd,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef net-3rd fill:#00bbbb,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef net-4th fill:#009999,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;
    classDef net-l3 fill:#0099bb,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5;

    classDef networks fill:#cdd,stroke-width:0px
    class nets-1,nets-2,nets-local networks

    classDef servers stroke-width:3px
    class Server-1,Server-2,Server-3 servers
    style Firewall stroke:#333,stroke-width:3px
```