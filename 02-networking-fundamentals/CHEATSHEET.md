# Networking Fundamentals Cheatsheet

## Quick Reference Guide for Platform Engineers

### OSI Model & TCP/IP Stack
```
OSI Model (7 Layers)          TCP/IP Model (4 Layers)      Protocols
----------------              --------------------         ---------
7. Application     ─┐
6. Presentation     ├────────> Application                HTTP, HTTPS, FTP, SSH, DNS
5. Session         ─┘
4. Transport       ────────> Transport                   TCP, UDP
3. Network         ────────> Internet                    IP, ICMP, ARP
2. Data Link       ─┐
1. Physical        ─└────────> Network Access            Ethernet, Wi-Fi
```

### IP Addressing
```bash
# IPv4 Classes (Legacy - now using CIDR)
Class A: 1.0.0.0 - 126.255.255.255      (/8)  - 16M hosts
Class B: 128.0.0.0 - 191.255.255.255    (/16) - 65K hosts
Class C: 192.0.0.0 - 223.255.255.255    (/24) - 254 hosts

# Private IP Ranges (RFC 1918)
10.0.0.0/8          10.0.0.0 - 10.255.255.255
172.16.0.0/12       172.16.0.0 - 172.31.255.255
192.168.0.0/16      192.168.0.0 - 192.168.255.255

# Special Addresses
127.0.0.0/8         Loopback (localhost)
169.254.0.0/16      Link-local (APIPA)
224.0.0.0/4         Multicast
0.0.0.0             Default route

# CIDR Notation
/32 = 255.255.255.255    (1 host)
/24 = 255.255.255.0      (254 hosts)
/16 = 255.255.0.0        (65,534 hosts)
/8  = 255.0.0.0          (16,777,214 hosts)
```

### Network Commands
```bash
# Interface management (modern)
ip addr show                          # Show all IP addresses
ip addr add 192.168.1.10/24 dev eth0 # Add IP to interface
ip addr del 192.168.1.10/24 dev eth0 # Remove IP from interface
ip link show                         # Show network interfaces
ip link set eth0 up                  # Bring interface up
ip link set eth0 down                # Bring interface down

# Interface management (legacy)
ifconfig                             # Show network configuration
ifconfig eth0 192.168.1.10          # Set IP address
ifconfig eth0 up                     # Bring interface up
ifconfig eth0 down                   # Bring interface down

# Routing
ip route show                        # Show routing table
ip route add default via 192.168.1.1 # Add default gateway
ip route add 10.0.0.0/8 via 192.168.1.254 # Add static route
ip route del 10.0.0.0/8             # Delete route
route -n                             # Legacy routing table

# Network statistics
ip -s link                           # Interface statistics
netstat -i                           # Interface statistics (legacy)
```

### DNS Operations
```bash
# DNS lookups
dig example.com                      # Detailed DNS query
dig example.com +short               # Brief answer only
dig @8.8.8.8 example.com            # Query specific DNS server
dig example.com ANY                  # All DNS records
dig -x 8.8.8.8                      # Reverse DNS lookup
dig example.com MX                   # Mail exchange records
dig example.com NS                   # Name server records
dig example.com TXT                  # TXT records

# Alternative tools
nslookup example.com                 # Interactive DNS lookup
host example.com                     # Simple DNS lookup
host -t MX example.com              # Specific record type

# DNS configuration
cat /etc/resolv.conf                 # DNS resolver configuration
cat /etc/hosts                       # Local hostname resolution
cat /etc/nsswitch.conf              # Name service switch config

# DNS troubleshooting
dig +trace example.com               # Trace DNS resolution path
```

### Port and Service Testing
```bash
# Check listening ports (modern)
ss -tuln                             # All listening TCP/UDP ports
ss -tulpn                            # With process information
ss -tunp                             # All TCP/UDP connections
ss -t state established              # Established TCP connections
ss -o state established              # With timer information

# Check listening ports (legacy)
netstat -tuln                        # All listening ports
netstat -tulpn                       # With process information
netstat -antp                        # All TCP connections

# Check specific port
lsof -i :80                          # What's using port 80
lsof -i tcp:80                       # TCP port 80 specifically
fuser 80/tcp                         # Process using TCP port 80

# Test connectivity
nc -zv example.com 80                # Test if port 80 is open
nc -zvw3 example.com 22              # With 3 second timeout
telnet example.com 80                # Interactive connection test
timeout 5 bash -c '</dev/tcp/example.com/80' # Pure bash test

# Port scanning (be careful - only on your own systems)
nmap -p 80,443 example.com          # Scan specific ports
nmap -p- example.com                # Scan all ports
nmap -sV example.com                # Service version detection
```

### Network Diagnostics
```bash
# Connectivity testing
ping -c 4 example.com               # Send 4 ICMP echo requests
ping -c 4 -i 2 example.com          # With 2 second interval
ping -c 4 -s 1500 example.com       # With specific packet size

# Path tracing
traceroute example.com              # Trace route to host
traceroute -n example.com           # Don't resolve hostnames
mtr example.com                     # Combined ping/traceroute
mtr -r -c 10 example.com            # Report mode (10 cycles)

# Network capture
tcpdump -i eth0                     # Capture on interface
tcpdump -i eth0 port 80             # Capture HTTP traffic
tcpdump -i eth0 -w capture.pcap     # Write to file
tcpdump -r capture.pcap             # Read from file
tcpdump -i eth0 -c 100              # Capture 100 packets
tcpdump -i eth0 host 192.168.1.1    # Specific host
tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0' # SYN packets

# ARP operations
ip neigh show                       # Show ARP cache (modern)
arp -a                              # Show ARP cache (legacy)
arp -d 192.168.1.1                  # Delete ARP entry
ip neigh flush all                  # Clear ARP cache
```

### Firewall Management

#### iptables
```bash
# View rules
iptables -L                         # List all rules
iptables -L -n -v                   # Verbose with packet counts
iptables -L INPUT -n                # Specific chain
iptables -S                         # Show rules as commands

# Basic rules
iptables -A INPUT -p tcp --dport 80 -j ACCEPT    # Allow HTTP
iptables -A INPUT -p tcp --dport 22 -j ACCEPT    # Allow SSH
iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT    # Allow subnet
iptables -A INPUT -j DROP                         # Default deny

# Save and restore
iptables-save > /etc/iptables/rules.v4           # Save rules
iptables-restore < /etc/iptables/rules.v4        # Restore rules

# Delete rules
iptables -D INPUT 3                 # Delete rule 3 from INPUT
iptables -F                         # Flush all rules (DANGER!)
```

#### firewalld (RHEL/CentOS)
```bash
# Status and info
firewall-cmd --state                # Check if running
firewall-cmd --list-all             # List all settings
firewall-cmd --list-services        # List allowed services
firewall-cmd --list-ports           # List allowed ports

# Allow services/ports
firewall-cmd --add-service=http     # Temporary
firewall-cmd --add-service=http --permanent  # Permanent
firewall-cmd --add-port=8080/tcp --permanent # Allow port
firewall-cmd --reload               # Apply permanent changes

# Remove rules
firewall-cmd --remove-service=http --permanent
firewall-cmd --remove-port=8080/tcp --permanent
```

#### ufw (Ubuntu)
```bash
# Basic operations
ufw status                          # Check status
ufw enable                          # Enable firewall
ufw disable                         # Disable firewall

# Allow/Deny
ufw allow 22/tcp                    # Allow SSH
ufw allow 80/tcp                    # Allow HTTP
ufw allow from 192.168.1.0/24       # Allow subnet
ufw deny 23/tcp                     # Deny telnet

# Delete rules
ufw status numbered                 # Show rule numbers
ufw delete 2                        # Delete rule 2

# Reset
ufw reset                           # Reset to default
```

### Network File Transfer
```bash
# wget
wget https://example.com/file.zip   # Download file
wget -c https://example.com/file    # Continue partial download
wget -r https://example.com         # Recursive download
wget --limit-rate=200k url          # Limit download speed

# curl
curl -O https://example.com/file    # Download (keep name)
curl -o local.zip https://example.com/file # Download (rename)
curl -I https://example.com         # Headers only
curl -L https://example.com         # Follow redirects
curl -u user:pass ftp://example.com # With authentication
curl -X POST -d "data" url          # POST request

# scp (secure copy)
scp file.txt user@host:/path        # Copy to remote
scp user@host:/path/file.txt .      # Copy from remote
scp -r directory user@host:/path    # Copy directory
scp -P 2222 file.txt user@host:/path # Non-standard port

# rsync (efficient sync)
rsync -av source/ dest/             # Archive mode, verbose
rsync -avz source/ user@host:/dest/ # With compression
rsync -av --delete source/ dest/    # Delete extra files
rsync -av --progress source/ dest/  # Show progress
rsync -av --exclude='*.log' src/ dst/ # Exclude pattern
```

### SSL/TLS Testing
```bash
# Check SSL certificate
openssl s_client -connect example.com:443 # Connect to SSL
openssl s_client -showcerts -connect example.com:443 # Show certs
echo | openssl s_client -connect example.com:443 2>/dev/null | openssl x509 -noout -dates # Expiry dates

# Certificate information
openssl x509 -in cert.pem -text -noout # View certificate
openssl x509 -in cert.pem -noout -fingerprint # Fingerprint
openssl x509 -in cert.pem -noout -subject # Subject

# Test specific TLS version
openssl s_client -tls1_2 -connect example.com:443
openssl s_client -tls1_3 -connect example.com:443
```

### VPN and Tunneling
```bash
# SSH tunneling
ssh -L 8080:localhost:80 user@host  # Local port forward
ssh -R 8080:localhost:80 user@host  # Remote port forward
ssh -D 1080 user@host               # SOCKS proxy

# OpenVPN
openvpn --config client.ovpn        # Connect to VPN
systemctl status openvpn@client     # Check service status

# WireGuard
wg-quick up wg0                     # Start VPN
wg-quick down wg0                   # Stop VPN
wg show                             # Show status
```

### Network Performance Testing
```bash
# Bandwidth testing
iperf3 -s                           # Server mode
iperf3 -c server_ip                 # Client mode
iperf3 -c server_ip -t 30           # Test for 30 seconds
iperf3 -c server_ip -P 4            # 4 parallel streams

# Speed test
curl -s https://raw.githubusercontent.com/sivel/speedtest-cli/master/speedtest.py | python -

# Network load
nethogs                             # Per-process bandwidth monitor
iftop                               # Interface bandwidth monitor
iptraf-ng                           # Network statistics
```

### VLAN and Bridge Configuration
```bash
# VLAN (802.1Q)
ip link add link eth0 name eth0.10 type vlan id 10  # Create VLAN
ip addr add 192.168.10.1/24 dev eth0.10             # Add IP
ip link set dev eth0.10 up                          # Bring up

# Bridge
ip link add br0 type bridge         # Create bridge
ip link set eth0 master br0          # Add interface to bridge
ip link set eth1 master br0
ip link set br0 up                   # Bring up bridge
brctl show                           # Show bridges (legacy)
```

### Network Namespaces
```bash
# Create namespace
ip netns add myns                    # Create namespace
ip netns list                        # List namespaces

# Configure namespace
ip netns exec myns ip link set lo up # Bring up loopback
ip link add veth0 type veth peer name veth1 # Create veth pair
ip link set veth1 netns myns         # Move to namespace

# Execute in namespace
ip netns exec myns ping 8.8.8.8      # Run command in namespace
ip netns exec myns bash              # Shell in namespace

# Delete namespace
ip netns del myns                    # Delete namespace
```

## Common Network Ports

### Well-Known Ports (0-1023)
```
20/21   FTP (File Transfer Protocol)
22      SSH (Secure Shell)
23      Telnet
25      SMTP (Email)
53      DNS (Domain Name System)
67/68   DHCP
80      HTTP
110     POP3 (Email)
123     NTP (Network Time Protocol)
143     IMAP (Email)
443     HTTPS (HTTP Secure)
465     SMTPS (SMTP Secure)
514     Syslog
587     SMTP (Submission)
993     IMAPS (IMAP Secure)
995     POP3S (POP3 Secure)
```

### Registered Ports (1024-49151)
```
1433    MS SQL Server
1521    Oracle Database
3306    MySQL
3389    RDP (Remote Desktop)
5432    PostgreSQL
5672    AMQP (RabbitMQ)
5900    VNC
6379    Redis
8080    HTTP Alternate
8443    HTTPS Alternate
9090    Prometheus
9200    Elasticsearch
27017   MongoDB
```

## Subnetting Quick Reference

### CIDR to Subnet Mask
```
/8  = 255.0.0.0          = 16,777,214 hosts
/16 = 255.255.0.0        = 65,534 hosts
/24 = 255.255.255.0      = 254 hosts
/25 = 255.255.255.128    = 126 hosts
/26 = 255.255.255.192    = 62 hosts
/27 = 255.255.255.224    = 30 hosts
/28 = 255.255.255.240    = 14 hosts
/29 = 255.255.255.248    = 6 hosts
/30 = 255.255.255.252    = 2 hosts (point-to-point)
/32 = 255.255.255.255    = 1 host (single IP)
```

## Protocol Quick Reference

### TCP vs UDP
```
TCP (Transmission Control Protocol)
- Connection-oriented
- Reliable delivery
- Flow control
- Error checking
- Used for: HTTP, HTTPS, SSH, FTP, email

UDP (User Datagram Protocol)
- Connectionless
- Best-effort delivery
- No flow control
- Faster than TCP
- Used for: DNS, DHCP, streaming, gaming, VoIP
```

### HTTP Status Codes
```
1xx - Informational
2xx - Success (200 OK, 201 Created, 204 No Content)
3xx - Redirection (301 Moved, 302 Found, 304 Not Modified)
4xx - Client Error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found)
5xx - Server Error (500 Internal Server Error, 502 Bad Gateway, 503 Service Unavailable)
```

## Troubleshooting Workflow

### Network Connectivity Issues
```bash
# 1. Check local interface
ip addr show
ip link show

# 2. Check local routing
ip route show

# 3. Test local connectivity
ping 127.0.0.1          # Loopback
ping <local_ip>          # Local interface

# 4. Test gateway
ping <gateway_ip>

# 5. Test DNS
ping 8.8.8.8            # Google DNS (by IP)
ping google.com         # By name (tests DNS)

# 6. Check DNS resolution
dig google.com

# 7. Trace route
traceroute google.com

# 8. Check firewall
iptables -L -n
ufw status

# 9. Check listening services
ss -tuln

# 10. Capture traffic
tcpdump -i eth0 -n
```

## Best Practices

1. **Always test in development** before applying network changes in production
2. **Document network changes** including date, reason, and rollback plan
3. **Use persistent configuration** instead of temporary commands
4. **Monitor network performance** regularly to establish baselines
5. **Implement redundancy** for critical network paths
6. **Segment networks** for security (VLANs, subnets)
7. **Use SSH keys** instead of passwords for authentication
8. **Keep firewall rules minimal** and well-documented
9. **Regular security audits** of open ports and services
10. **Have rollback plans** for configuration changes

## Pro Tips

- Use `ip` commands instead of legacy `ifconfig`/`route` where possible
- `ss` is faster and more feature-rich than `netstat`
- Combine `watch` with network commands for real-time monitoring: `watch -n 1 'ss -s'`
- Use `mtr` instead of separate ping/traceroute for better diagnostics
- Learn tcpdump filters to capture only relevant traffic
- Keep a network diagram updated as infrastructure changes
- Use configuration management tools for network consistency
- Test firewall rules thoroughly before applying to production
