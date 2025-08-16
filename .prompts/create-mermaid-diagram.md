# Create a Mermaid diagram for Kubernetes networking architecture
I have a networking architecture description with the following components and relationships:

[PASTE YOUR NETWORKING DESCRIPTION HERE]

The description includes:
- Network interfaces and their IP addresses/subnets
- How interfaces are connected (bridged, enslaved, etc.)
- The layered structure (e.g., VM inside Pod, Pod on Host)
- Network namespaces and routing

Please create a Mermaid diagram that shows:
- The layered architecture (VM → Pod → Host) as nested subgraphs
- All network interfaces with their IP addresses
- The connections between interfaces with descriptive labels
- Proper styling to distinguish different layers
- Edge labels properly delimited with | characters on both sides

Use this format for edge labels: source -->|"description"| target

The diagram should clearly illustrate the network topology and how traffic flows between the different layers and interfaces.