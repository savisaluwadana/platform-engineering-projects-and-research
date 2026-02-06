# Networking Fundamentals

## Theory

### Core Concepts

#### OSI Model: The Seven-Layer Framework
The Open Systems Interconnection (OSI) model is a conceptual framework for understanding network communications:

1. **Physical Layer**: Deals with raw bit transmission over physical media (cables, radio waves). Defines voltage levels, pin layouts, and physical data rates.
2. **Data Link Layer**: Provides node-to-node data transfer and handles error correction from the physical layer. Includes MAC addresses and protocols like Ethernet and Wi-Fi.
3. **Network Layer**: Manages packet forwarding and routing through intermediate routers. IP (Internet Protocol) operates at this layer, handling logical addressing and path determination.
4. **Transport Layer**: Ensures complete data transfer with error recovery and flow control. TCP provides reliable, ordered delivery while UDP offers lightweight, connectionless communication.
5. **Session Layer**: Manages sessions and controls dialogues between computers, handling connection establishment, maintenance, and termination.
6. **Presentation Layer**: Translates data between application and network formats, handling encryption, compression, and encoding.
7. **Application Layer**: Provides network services directly to end-user applications (HTTP, FTP, SMTP, DNS).

Understanding the OSI model helps diagnose network issues by isolating problems to specific layers and choosing appropriate troubleshooting tools.

#### TCP/IP Model: The Internet Protocol Suite
The TCP/IP model is a more practical, condensed framework used in real-world networking:

- **Network Interface Layer**: Combines OSI's Physical and Data Link layers, handling hardware addressing and physical transmission.
- **Internet Layer**: Routes packets across networks using IP addressing, including IPv4 and IPv6 protocols, ICMP for diagnostics, and routing protocols.
- **Transport Layer**: Provides end-to-end communication services with TCP (connection-oriented, reliable) and UDP (connectionless, fast).
- **Application Layer**: Encompasses OSI's Session, Presentation, and Application layers, including protocols like HTTP, DNS, SMTP, and SSH.

The TCP/IP model reflects how the internet actually works, making it essential for practical network engineering.

#### IP Addressing and Subnetting
IP addresses uniquely identify devices on networks:

- **IPv4**: 32-bit addresses written as four octets (e.g., 192.168.1.1). Provides ~4.3 billion addresses, now largely exhausted.
- **IPv6**: 128-bit addresses using hexadecimal notation (e.g., 2001:0db8:85a3::8a2e:0370:7334). Provides virtually unlimited addresses.
- **CIDR Notation**: Classless Inter-Domain Routing expresses IP ranges using slash notation (e.g., 192.168.1.0/24 represents 256 addresses).
- **Subnetting**: Dividing networks into smaller subnetworks for better organization, security, and efficient IP address utilization.
- **Public vs Private IPs**: Public IPs are globally routable; private IPs (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16) are used internally with NAT for internet access.

Subnetting skills enable platform engineers to design efficient network architectures and implement proper network segmentation for security.

#### Network Protocols
Protocols define rules for network communication:

- **TCP (Transmission Control Protocol)**: Connection-oriented, reliable delivery with error checking, flow control, and congestion management. Used for applications requiring guaranteed delivery (HTTP, SSH, email).
- **UDP (User Datagram Protocol)**: Connectionless, low-latency protocol without delivery guarantees. Ideal for real-time applications (DNS, streaming, gaming).
- **ICMP (Internet Control Message Protocol)**: Diagnostic and error-reporting protocol used by ping and traceroute.
- **HTTP/HTTPS**: Application protocols for web communication. HTTPS adds SSL/TLS encryption for secure data transfer.
- **DNS (Domain Name System)**: Translates human-readable domain names to IP addresses, functioning as the internet's phone book.
- **DHCP (Dynamic Host Configuration Protocol)**: Automatically assigns IP addresses and network configuration to devices.
- **ARP (Address Resolution Protocol)**: Maps IP addresses to MAC addresses on local networks.

Protocol knowledge helps in choosing the right communication method for different application requirements and troubleshooting network issues.

#### Network Devices
Understanding network hardware is crucial for infrastructure design:

- **Routers**: Layer 3 devices that forward packets between different networks based on IP addresses, making routing decisions using routing tables.
- **Switches**: Layer 2 devices that forward frames within a network based on MAC addresses, creating separate collision domains for improved performance.
- **Firewalls**: Security devices that filter traffic based on rules, protecting networks from unauthorized access and attacks.
- **Load Balancers**: Distribute traffic across multiple servers to improve availability, performance, and reliability. Support various algorithms (round-robin, least connections, IP hash).
- **Proxies**: Intermediary servers that handle requests on behalf of clients, providing caching, security, and access control.

Platform engineers must understand these devices to design resilient, performant network architectures.

#### Network Security
Security is paramount in network design:

- **Firewalls**: Stateful inspection of packets, blocking malicious traffic while allowing legitimate communication. Can be network-based or host-based.
- **VPNs (Virtual Private Networks)**: Encrypted tunnels for secure remote access and site-to-site connectivity. Common protocols include OpenVPN, IPSec, and WireGuard.
- **SSL/TLS**: Cryptographic protocols securing data in transit, providing encryption, authentication, and data integrity for web traffic and other applications.
- **NAT (Network Address Translation)**: Maps private IP addresses to public ones, conserving IPv4 addresses and adding a layer of security by hiding internal network structure.
- **Network Segmentation**: Dividing networks into isolated segments using VLANs, subnets, and firewall rules to limit attack surface and contain breaches.
- **Zero Trust**: Security model assuming no implicit trust, requiring verification for every access request regardless of location.

Security knowledge ensures platform engineers can protect infrastructure from evolving threats.

#### DNS: The Internet's Directory
Domain Name System translates human-readable names to machine-readable IP addresses:

- **DNS Resolution Process**: Recursive queries from client to DNS resolver to root servers to TLD servers to authoritative nameservers.
- **DNS Records**:
  - **A/AAAA**: Map domain names to IPv4/IPv6 addresses
  - **CNAME**: Create aliases pointing to other domain names
  - **MX**: Specify mail servers for email delivery
  - **TXT**: Store arbitrary text data, often used for verification and security (SPF, DKIM, DMARC)
  - **NS**: Designate authoritative nameservers for a domain
  - **SOA**: Provide authoritative information about a DNS zone
- **DNS Caching**: Improves performance by storing results temporarily based on TTL (Time To Live).
- **DNS Security**: DNSSEC adds cryptographic signatures to prevent cache poisoning and ensure authenticity.

DNS is critical infrastructure - understanding it enables proper configuration, troubleshooting, and security implementation.

### Key Skills
1. **Design and implement network architectures**: Plan scalable, resilient network topologies considering performance, security, and cost. Understand when to use different network patterns and topologies.
2. **Troubleshoot network connectivity issues**: Systematically diagnose problems using tools like ping, traceroute, tcpdump, and Wireshark. Understand the OSI model to isolate issues to specific layers.
3. **Configure firewalls and security groups**: Implement network security policies using iptables, cloud security groups, and network ACLs. Balance security with operational requirements.
4. **Understand routing and switching**: Configure routing protocols (static, OSPF, BGP), VLANs, and switch configurations. Optimize traffic flow and prevent loops.
5. **Implement load balancing and high availability**: Design systems that distribute traffic effectively, eliminate single points of failure, and maintain service during component failures.

## Projects

### Beginner Level

#### Project 1: Network Scanner Tool
**Objective**: Build a network discovery and scanning tool
- Scan IP ranges for active hosts
- Detect open ports on hosts
- Identify services running on ports
- Generate network topology map
- Export results to CSV/JSON

**Learning Outcomes**: Network protocols, port scanning, service detection

#### Project 2: Simple HTTP Load Balancer
**Objective**: Create a basic round-robin load balancer
- Implement round-robin algorithm
- Health checks for backend servers
- Request logging
- Sticky sessions support
- Metrics collection

**Learning Outcomes**: Load balancing concepts, TCP/HTTP, health monitoring

#### Project 3: DNS Query Tool
**Objective**: Build a DNS lookup and analysis tool
- Query different DNS record types
- Trace DNS resolution path
- DNS cache implementation
- Performance metrics
- Support for custom DNS servers

**Learning Outcomes**: DNS protocol, record types, caching strategies

### Intermediate Level

#### Project 4: Network Traffic Analyzer
**Objective**: Create a packet capture and analysis tool
- Capture network packets using libpcap/tcpdump
- Parse common protocols (HTTP, DNS, TCP, UDP)
- Filter traffic by protocol, port, IP
- Generate traffic statistics
- Detect anomalies

**Learning Outcomes**: Packet analysis, protocol understanding, Wireshark basics

#### Project 5: VPN Implementation
**Objective**: Set up a VPN server and client
- OpenVPN or WireGuard setup
- Certificate management
- Client configuration automation
- Traffic routing and NAT
- Monitoring and logging

**Learning Outcomes**: VPN protocols, encryption, tunneling, routing

#### Project 6: Software-Defined Network (SDN) Controller
**Objective**: Build a basic SDN controller
- OpenFlow protocol implementation
- Switch management
- Flow rule programming
- Network topology visualization
- Traffic engineering

**Learning Outcomes**: SDN concepts, OpenFlow, network programmability

### Advanced Level

#### Project 7: Multi-Region Service Mesh
**Objective**: Implement a service mesh for microservices
- Service discovery and registration
- Load balancing with multiple algorithms
- Circuit breaker pattern
- Distributed tracing
- mTLS for service-to-service communication
- Traffic shaping and canary deployments

**Learning Outcomes**: Service mesh architecture, microservices networking, security

#### Project 8: DDoS Protection System
**Objective**: Build a DDoS detection and mitigation system
- Traffic pattern analysis
- Anomaly detection using ML
- Rate limiting implementation
- IP blacklisting/whitelisting
- Integration with cloud WAF
- Real-time alerting

**Learning Outcomes**: DDoS attacks, mitigation strategies, security

#### Project 9: Network Automation Framework
**Objective**: Create a network device automation platform
- Multi-vendor device support (Cisco, Juniper, etc.)
- NETCONF/RESTCONF implementation
- Configuration templating
- Compliance checking
- Change management workflow
- Automated testing and rollback

**Learning Outcomes**: Network automation, NETCONF, configuration management

## Resources

### Documentation
- [Cisco Networking Basics](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13769-5.html)
- [RFC Index](https://www.rfc-editor.org/)
- [TCP/IP Guide](http://www.tcpipguide.com/)

### Books
- "Computer Networking: A Top-Down Approach" by Kurose & Ross
- "TCP/IP Illustrated" by W. Richard Stevens
- "Network Warrior" by Gary Donahue

### Tools
- Wireshark
- nmap
- tcpdump
- iperf
- netcat

### Certifications
- Cisco CCNA
- CompTIA Network+
- Juniper JNCIA
