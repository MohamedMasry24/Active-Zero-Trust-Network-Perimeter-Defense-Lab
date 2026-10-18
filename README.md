# Active-Zero-Trust-Network-Perimeter-Defense-Lab

In this project, we are making a zero-trust virtual network.

First, what is Zero Trust? 

Every device or user must prove they are allowed to access a resource. Being connected to the company network doesn't automatically mean you can access everything.

Zero Trust operates on 3 principles: 
1. Principle of least privilege 
Never Trust, Always Verify 
3. Assume everything is a data breach

So in this project, we are making a fully virtualized network. Instead of relying on traditional, flat network topologies where everything trusts everything, we will enforce continuous verification, strict microsegmentation, deep packet inspection, and explicit access controls across multiple isolated security zones.

Our technical objectives: 

-Strict Microsegmentation: We will construct physically and logically isolated network domains using Virtual network infrastructure.
-Least-Privilege Inter-Zone Access Controls: Replace implicit allow rules with deterministic firewall policies permitting only explicitly documented ports, protocols, and destination hosts.
-Deep Packet Inspection: Deploy signature-based and behavioral IDS/IPS rules to actively block malicious traffic, exploit payloads, and unauthorized scanning.
-Attack Simulation & Defensive Validation: Test the network's security by simulating attacks from untrusted and user networks to check whether security rules block unauthorized access and security alerts work correctly.

For this project, we will need to use **OPNsense**
