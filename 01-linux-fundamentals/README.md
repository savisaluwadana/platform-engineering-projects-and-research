# Linux Fundamentals

## Theory

### Core Concepts

#### Linux Architecture
Linux is a Unix-like operating system built on a layered architecture:

- **Kernel**: The core of the operating system that manages hardware resources, processes, memory, and system calls. The Linux kernel is monolithic but supports loadable kernel modules (LKMs) for extending functionality without rebooting.
- **Shell**: The command-line interface (CLI) that interprets user commands and communicates with the kernel. Common shells include Bash, Zsh, and Fish, each offering different features for scripting and interactive use.
- **System Libraries**: Shared libraries (like glibc) that provide standard functions for applications, enabling code reuse and reducing binary sizes.
- **System Utilities**: Essential programs for system administration, file manipulation, and user management, found in directories like `/bin`, `/sbin`, and `/usr/bin`.

Understanding this architecture helps platform engineers optimize system performance, troubleshoot issues at the right layer, and make informed decisions about kernel tuning and system configuration.

#### File System Hierarchy Standard (FHS)
Linux follows the Filesystem Hierarchy Standard, organizing files in a logical structure:

- **`/etc`**: System configuration files and scripts. Critical for understanding how services are configured.
- **`/var`**: Variable data including logs (`/var/log`), temporary files, and application data. Essential for monitoring and troubleshooting.
- **`/home`**: User home directories containing personal files and configurations.
- **`/usr`**: User programs and data, including applications (`/usr/bin`), libraries (`/usr/lib`), and documentation.
- **`/opt`**: Optional third-party software packages.
- **`/bin`** and **`/sbin`**: Essential command binaries for system boot and repair.
- **`/tmp`**: Temporary files, often cleared on reboot.
- **`/proc`** and **`/sys`**: Virtual filesystems providing interfaces to kernel data structures and hardware information.

Mastering the FHS enables efficient navigation, proper application deployment, and effective troubleshooting.

#### Users & Permissions
Linux implements a robust security model based on users, groups, and permissions:

- **UID/GID**: User ID and Group ID are numeric identifiers for users and groups. UID 0 is reserved for root (superuser).
- **chmod**: Change file mode bits to set read (r=4), write (w=2), and execute (x=1) permissions for owner, group, and others.
- **chown**: Change file ownership to different users or groups.
- **umask**: Default permission mask that determines initial permissions for newly created files and directories.
- **sudo**: Execute commands with elevated privileges without logging in as root, promoting security through the principle of least privilege.
- **ACLs (Access Control Lists)**: Extended permissions beyond traditional Unix permissions, allowing fine-grained access control for specific users or groups.

Proper permission management is crucial for system security, preventing unauthorized access while enabling legitimate users to perform their tasks.

#### Process Management
Processes are running instances of programs, and managing them is fundamental to system administration:

- **ps**: Display information about active processes, showing PID, CPU usage, memory consumption, and command details.
- **top/htop**: Interactive process viewers showing real-time system resource usage and process statistics.
- **systemctl**: Control the systemd init system, managing services (start, stop, enable, disable), viewing logs, and checking service status.
- **Services**: Long-running background processes that provide system functionality (web servers, databases, etc.).
- **Daemons**: Background processes that start at boot and run continuously, typically named with a 'd' suffix (sshd, httpd).
- **Process States**: Understanding running, sleeping, stopped, and zombie states helps diagnose system issues.
- **Signals**: Inter-process communication mechanisms (SIGTERM, SIGKILL, SIGHUP) used to control process behavior.

Effective process management ensures optimal resource utilization and system stability.

#### Package Management
Package managers automate software installation, updates, and dependency resolution:

- **APT (Advanced Package Tool)**: Debian/Ubuntu package manager with commands like `apt-get`, `apt-cache`, and the modern `apt`.
- **YUM/DNF**: Red Hat/CentOS/Fedora package managers, with DNF being the modern replacement for YUM.
- **rpm/dpkg**: Low-level package managers that work with individual package files (.rpm, .deb).
- **Package Repositories**: Centralized servers hosting software packages, configured in `/etc/apt/sources.list` or `/etc/yum.repos.d/`.
- **Dependency Resolution**: Automatic handling of package dependencies, ensuring all required libraries and tools are installed.
- **Version Pinning**: Locking packages to specific versions to ensure system stability.

Understanding package management is essential for maintaining consistent environments and managing software deployments at scale.

#### Shell Scripting
Bash scripting automates repetitive tasks and enables infrastructure as code:

- **Variables**: Store and manipulate data (`VAR=value`, `$VAR` for expansion).
- **Conditionals**: Make decisions with `if`, `elif`, `else`, and test conditions (`[ ]`, `[[ ]]`).
- **Loops**: Iterate with `for`, `while`, and `until` loops.
- **Functions**: Organize reusable code blocks with parameters and return values.
- **Command Substitution**: Capture command output (`$(command)` or backticks).
- **Exit Codes**: Use `$?` to check if commands succeeded (0) or failed (non-zero).
- **Redirection and Pipes**: Redirect input/output (`>`, `>>`, `<`) and chain commands (`|`).
- **Error Handling**: Implement robust error checking with `set -e`, `set -u`, and `trap`.

Shell scripting is the foundation of system automation, enabling repeatable, version-controlled operations.

#### Networking Fundamentals
Linux networking tools and concepts are essential for platform engineering:

- **ifconfig/ip**: Configure network interfaces, assign IP addresses, and manage routes. The `ip` command is the modern replacement for `ifconfig`.
- **netstat/ss**: Display network connections, routing tables, and interface statistics. `ss` is faster and more feature-rich than `netstat`.
- **DNS Resolution**: Configure DNS servers in `/etc/resolv.conf` and understand the resolution process through `/etc/nsswitch.conf`.
- **Routing**: Configure routing tables with `ip route` to control packet forwarding between networks.
- **Network Namespaces**: Isolate network stacks for containers and virtual environments.
- **Firewall**: Manage firewall rules with `iptables`, `nftables`, or `firewalld`.
- **Network Troubleshooting**: Use `ping`, `traceroute`, `tcpdump`, and `wireshark` to diagnose connectivity issues.

Network knowledge enables platform engineers to design resilient architectures, troubleshoot connectivity problems, and implement security policies.

### Key Skills
1. **Navigate and manage file systems efficiently**: Master file operations, understand the FHS, and use tools like `find`, `grep`, and `tree` to locate and manipulate files quickly.
2. **Understand and configure user permissions**: Implement security best practices through proper user management, permission assignment, and the principle of least privilege.
3. **Monitor and troubleshoot system resources**: Use monitoring tools to identify bottlenecks, analyze performance metrics, and optimize resource allocation.
4. **Write automation scripts**: Develop robust shell scripts that automate routine tasks, reduce human error, and enable infrastructure as code practices.
5. **Configure network interfaces and troubleshoot connectivity**: Set up networking, diagnose issues using appropriate tools, and understand how data flows through your infrastructure.

## Projects

### Beginner Level

#### Project 1: System Administration Dashboard Script
**Objective**: Create a bash script that displays system information
- Display CPU usage, memory usage, disk space
- Show logged-in users
- List top 10 processes by CPU/memory
- Add color-coded alerts for high usage (>80%)

**Learning Outcomes**: Basic bash scripting, system commands, process monitoring

#### Project 2: User Management Automation
**Objective**: Build scripts to automate user provisioning
- Create users with specific groups and home directories
- Set up SSH keys automatically
- Generate random passwords
- Create user audit reports

**Learning Outcomes**: User management, file permissions, security basics

#### Project 3: Log Rotation and Cleanup Tool
**Objective**: Implement custom log rotation
- Scan directories for log files
- Compress logs older than X days
- Delete logs older than Y days
- Send email notifications on errors

**Learning Outcomes**: Cron jobs, file operations, compression, email setup

### Intermediate Level

#### Project 4: Service Health Monitor
**Objective**: Create a monitoring system for critical services
- Check if services are running (nginx, MySQL, etc.)
- Auto-restart failed services
- Log restart attempts
- Send alerts via webhook/email
- Dashboard showing service status

**Learning Outcomes**: Systemd, service management, monitoring, automation

#### Project 5: Automated Backup System
**Objective**: Build a comprehensive backup solution
- Support multiple backup types (full, incremental, differential)
- Encrypt backups using GPG
- Upload to remote storage (S3, FTP, rsync)
- Verify backup integrity
- Restoration testing automation

**Learning Outcomes**: Backup strategies, encryption, remote storage, scheduling

#### Project 6: Custom Package Repository
**Objective**: Set up a local package repository
- Create APT/YUM repository
- Package custom applications
- Implement repository mirroring
- Set up automatic updates

**Learning Outcomes**: Package management, repository setup, distribution

### Advanced Level

#### Project 7: Multi-Server Configuration Management
**Objective**: Build a lightweight configuration management system
- SSH-based parallel command execution
- Template-based configuration deployment
- Inventory management
- Idempotent operations
- Change tracking and rollback

**Learning Outcomes**: SSH automation, configuration management concepts, scripting

#### Project 8: Performance Monitoring and Tuning System
**Objective**: Create a comprehensive performance analysis tool
- Collect system metrics (CPU, memory, I/O, network)
- Store metrics in time-series database
- Analyze performance bottlenecks
- Generate optimization recommendations
- Automated tuning for kernel parameters

**Learning Outcomes**: Performance analysis, metrics collection, system tuning

#### Project 9: Security Hardening Automation
**Objective**: Automate CIS security benchmarks
- Scan for security vulnerabilities
- Implement security hardening (firewall, SELinux, AppArmor)
- File integrity monitoring
- Audit log analysis
- Generate compliance reports

**Learning Outcomes**: Security best practices, compliance, auditing

## Resources

### Documentation
- [Linux Documentation Project](https://tldp.org/)
- [Arch Linux Wiki](https://wiki.archlinux.org/)
- [Red Hat Documentation](https://access.redhat.com/documentation/)

### Books
- "The Linux Command Line" by William Shotts
- "Linux Bible" by Christopher Negus
- "How Linux Works" by Brian Ward

### Practice Platforms
- Linux Foundation Training
- OverTheWire Bandit
- HackerRank Linux Shell

### Certifications
- Linux Foundation Certified System Administrator (LFCS)
- Red Hat Certified System Administrator (RHCSA)
- CompTIA Linux+
