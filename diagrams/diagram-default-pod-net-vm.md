# Virtual Machine Networking Diagram

```mermaid
graph TB
    subgraph "Kubernetes Cluster"
        subgraph "Pod Network (10.131.0.0/23)"
            VM_POD[VM Pod<br/>10.131.1.97/23]
        end

        subgraph "VM Internal Network (10.0.2.0/24)"
            VM[Virtual Machine<br/>10.0.2.2/24]
            VM_GW[VM Gateway<br/>10.0.2.1]
        end
    end

    subgraph "External Network"
        INTERNET[Internet]
    end

    %% Connections
    VM_POD -->|"Hosts"| VM
    VM -->|"Default Route"| VM_GW
    VM_GW -->|"Masquerades as"| VM_POD
    VM_POD -->|"Cluster Network"| INTERNET

    %% Styling
    classDef podStyle fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef vmStyle fill:#f3e5f5,stroke:#4a148c,stroke-width:2px
    classDef gatewayStyle fill:#fff3e0,stroke:#e65100,stroke-width:2px
    classDef networkStyle fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px

    class VM_POD podStyle
    class VM vmStyle
    class VM_GW gatewayStyle
    class INTERNET networkStyle
```

## Key Points

- **VM Internal IP**: `10.0.2.2/24` (always static)
- **Pod IP**: `10.131.1.97/23` (assigned by cluster)
- **VM Gateway**: `10.0.2.1` (handles routing)
- **Masquerading**: VM traffic appears to come from pod IP `10.131.1.97/23`
- **Network Isolation**: VM operates in separate `10.0.2.0/24` subnet within pod

## Network Flow

1. VM sends traffic to `10.0.2.1` (default gateway)
2. Gateway masquerades traffic to appear from pod IP `10.131.1.97/23`
3. Traffic flows through pod network to external destinations
4. Return traffic follows reverse path through pod network
