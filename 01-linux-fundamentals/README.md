# Linux Fundamentals

## Theory

### Core Concepts
- **Linux Architecture**: Kernel, Shell, System Libraries, System Utilities
- **File System Hierarchy**: Understanding `/etc`, `/var`, `/home`, `/usr`, `/opt`, `/bin`, `/sbin`
- **Users & Permissions**: UID/GID, chmod, chown, umask, sudo, ACLs
- **Process Management**: ps, top, htop, systemctl, services, daemons
- **Package Management**: apt, yum, dnf, rpm, dpkg
- **Shell Scripting**: Bash scripting fundamentals, variables, loops, conditionals
- **Networking**: ifconfig, ip, netstat, ss, DNS, routing

### Key Skills
1. Navigate and manage file systems efficiently
2. Understand and configure user permissions
3. Monitor and troubleshoot system resources
4. Write automation scripts
5. Configure network interfaces and troubleshoot connectivity

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
