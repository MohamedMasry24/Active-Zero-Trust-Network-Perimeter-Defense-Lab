# Active-Zero-Trust-Network-Perimeter-Defense-Lab

In this project, we are making a zero-trust virtual network.

First, what is Zero Trust? 

Every device or user must prove they are allowed to access a resource. Being connected to the company network doesn't automatically mean you can access everything.

Zero Trust operates on 3 principles: 
1. Principle of least privilege 
2.Never Trust, Always Verify 
3. Assume everything is a data breach

So in this project, we are making a fully virtualized network. Instead of relying on traditional, flat network topologies where everything trusts everything, we will enforce continuous verification, strict microsegmentation, deep packet inspection, and explicit access controls across multiple isolated security zones.

Our technical objectives: 

-Strict Microsegmentation: We will construct physically and logically isolated network domains using Virtual network infrastructure.
-Least-Privilege Inter-Zone Access Controls: Replace implicit allow rules with deterministic firewall policies permitting only explicitly documented ports, protocols, and destination hosts.
-Deep Packet Inspection: Deploy signature-based and behavioral IDS/IPS rules to actively block malicious traffic, exploit payloads, and unauthorized scanning.
-Attack Simulation & Defensive Validation: Test the network's security by simulating attacks from untrusted and user networks to check whether security rules block unauthorized access and security alerts work correctly.

For this project, we will need to use **OPNsense**

What is OPNsense? 

 OPNsense is deployed as our all-in-one network security gateway, functioning simultaneously as a Stateful Firewall, Intrusion Detection System (IDS), and Intrusion Prevention System (IPS) to monitor, filter, and actively protect our network environment.


## Network Architecture & Adapter Topology

To enforce strict microsegmentation and prevent lateral movement, the firewall interfaces are decoupled into discrete Layer 2 broadcast domains using VMware LAN Segments and host NAT.

**PHASE 1 ** 
In this phase, we will be provisioning the OPNsense virtual machine, and we configured four distinct network interfaces to eliminate flat network broadcast leakage.

**First interface** 
em0 --> WAN, which connects the firewall and authorized internal devices to the internet for updates, package downloads, etc. All unsolicited incoming connections are blocked, while permitted outgoing connections and their return traffic are allowed.

**Second interface**
em1 ---> User-LAN (Internal Client Zone): Contains employee computers and test devices. Users can access the internet and approved internal services, but they are blocked from accessing the firewall's administrative interface to prevent unauthorized changes.

**Third interface**
em2 ---> DMZ (Isolated Services Zone): Hosts services such as web servers that need to be accessible to users. Only approved connections are allowed, and DMZ devices cannot initiate connections to the internal user or management networks, helping prevent attackers from spreading if a server is compromised.

** Fourth interface**
em3 ---> SOC-MGMT (Security Management Zone): Dedicated to security administrators and monitoring tools. Only this zone can access the firewall's management interfaces, keeping administrative access separate from regular user traffic to reduce the risk of unauthorized access and attacks.

------------------------------------------------------

**PHASE 2**
Phase 2 focuses on establishing Client Access, Interface Re-Addressing, and Zero-Trust Baseline Rules.

For this phase, we need a virtual machine. Because the firewall's LAN is placed on an isolated VMware LAN segment, the host PC cannot access it. Attaching Kali Linux to "User-LAN" puts a machine directly inside the internal network to access the web management portal of OPNsense and later serve as our security testing machine.
1. So we changed the Kali Linux VM's network adapter to the User-LAN segment.
2. Booted Kali and automatically received an IP address from OPNsense via DHCP. Using the command: "sudo nmcli networking off && sudo nmcli networking on"
3. Verified connectivity to the OPNsense LAN gateway (192.168.1.1)
4. Logged into the OPNsense Web GUI to begin setup.


<img width="637" height="96" alt="image" src="https://github.com/user-attachments/assets/bd4e40ee-410f-4972-91f5-9ee02f8dff0c" />


1. Interface Assignment

After accessing the OPNsense Web GUI through Kali Linux, we configured the firewall's four interfaces with their final IP addresses:

em0 (WAN): Configured to obtain an IP address automatically through DHCP from VMware NAT.

em1 (User-LAN): Changed from the default 192.168.1.1/24 to the static IP 10.0.10.1/24.

em2 (DMZ): Enabled and assigned the static IP 10.0.20.1/24.

em3 (SOC-MGMT): Enabled and assigned the static IP 10.0.30.1/24.

Each internal interface now serves as the gateway for its own subnet, keeping the three network zones on separate IP networks.

2. DHCP and DNS Configuration

DHCP was enabled on all three internal networks so connected devices could automatically receive IP addresses.

Dynamic address range: .50 to .150 within each subnet.

Reserved range: .2 to .49, intended for servers and infrastructure requiring fixed IP addresses.

Expansion range: .151 to .254, left available for future devices.

The OPNsense DNS service was configured to listen on port 53 across the internal interfaces and forward DNS requests to upstream resolvers through the WAN. This allows internal devices to resolve domain names, such as google.com, into IP addresses.

3. Outbound Network Address Translation (NAT)

Automatic Source NAT was enabled to allow devices on the internal networks to access the internet through the firewall's WAN connection.

The configured internal subnets are:

User-LAN: 10.0.10.0/24

DMZ: 10.0.20.0/24

SOC-MGMT: 10.0.30.0/24

When internal devices send traffic to the internet, the firewall translates their source IP addresses to its WAN IP address. This allows multiple internal devices to share the same external connection.

4. Verification and Test Results

After configuration, we performed the following tests to confirm that the network services and connectivity were working correctly.

DHCP Test: Renewed the Kali Linux workstation's DHCP lease on eth0 and confirmed that it received an IP address from the User-LAN subnet (10.0.10.0/24).

DNS Test: Executed dig google.com @10.0.10.1. The query returned NOERROR with valid DNS answers, confirming that the firewall's DNS service was responding successfully.

Internet Connectivity and NAT Test: Used ping -c 3 8.8.8.8 and curl -I https://www.google.com to verify external connectivity. Successful results confirmed that outgoing traffic and its return traffic could pass through the firewall.

Gateway Connectivity Test: Sent ICMP ping requests to 10.0.20.1 (DMZ gateway) and 10.0.30.1 (SOC-MGMT gateway). Successful responses confirmed that these firewall interfaces were reachable from the test environment.

These tests confirmed that the initial network configuration, DHCP, DNS, outbound NAT, internet connectivity, and gateway reachability were functioning as expected.

