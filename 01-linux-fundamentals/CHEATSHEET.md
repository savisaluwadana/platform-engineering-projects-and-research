# Linux Fundamentals Cheatsheet

## Quick Reference Guide for Platform Engineers

### File System Navigation
```bash
# Basic navigation
pwd                    # Print working directory
ls -lah               # List all files with human-readable sizes
cd /path/to/dir       # Change directory
cd ~                  # Go to home directory
cd -                  # Go to previous directory

# Finding files
find /path -name "*.log"              # Find files by name
find /path -type f -mtime +7          # Find files older than 7 days
find /path -size +100M                # Find files larger than 100MB
locate filename                       # Quick file search (updatedb first)
which command                         # Find command location
whereis program                       # Find binary, source, manual

# Directory operations
mkdir -p /path/to/nested/dir          # Create nested directories
rmdir directory                       # Remove empty directory
rm -rf directory                      # Remove directory and contents (DANGER!)
tree -L 2                            # Show directory tree (2 levels)
```

### File Operations
```bash
# Viewing files
cat file.txt                          # Display entire file
less file.txt                         # Page through file (q to quit)
head -n 20 file.txt                   # First 20 lines
tail -n 20 file.txt                   # Last 20 lines
tail -f /var/log/syslog              # Follow file updates in real-time

# Editing files
nano file.txt                         # Simple text editor
vi file.txt                          # Powerful text editor
vim file.txt                         # Improved vi

# File manipulation
cp source dest                        # Copy file
cp -r source_dir dest_dir            # Copy directory recursively
mv source dest                        # Move/rename file
rm file.txt                          # Remove file
touch file.txt                       # Create empty file or update timestamp
```

### Permissions & Ownership
```bash
# Understanding permissions: rwxrwxrwx (owner-group-others)
# r=4, w=2, x=1

# Change permissions
chmod 755 file.sh                     # rwxr-xr-x (common for scripts)
chmod 644 file.txt                    # rw-r--r-- (common for files)
chmod +x file.sh                      # Add execute permission
chmod -R 755 directory                # Recursive permission change
chmod u+x,g+x,o-rwx file             # Add execute for user/group, remove all for others

# Change ownership
chown user:group file                 # Change owner and group
chown -R user:group directory         # Recursive ownership change
chgrp group file                      # Change group only

# Special permissions
chmod 4755 file                       # Set SUID (runs as owner)
chmod 2755 directory                  # Set SGID (inherit group)
chmod 1755 directory                  # Set sticky bit (only owner can delete)

# View permissions
ls -l file                           # Long format with permissions
stat file                            # Detailed file information
```

### Process Management
```bash
# Viewing processes
ps aux                               # All processes with details
ps aux | grep nginx                  # Find specific process
pgrep nginx                          # Get PID of process
top                                  # Interactive process viewer
htop                                 # Enhanced interactive viewer (install first)

# Process control
kill PID                             # Gracefully terminate (SIGTERM)
kill -9 PID                          # Force kill (SIGKILL)
killall process_name                 # Kill all processes by name
pkill pattern                        # Kill processes matching pattern

# Background jobs
command &                            # Run in background
jobs                                 # List background jobs
fg %1                               # Bring job to foreground
bg %1                               # Resume job in background
nohup command &                      # Run immune to hangups
```

### System Services (systemd)
```bash
# Service management
systemctl status service             # Check service status
systemctl start service              # Start service
systemctl stop service               # Stop service
systemctl restart service            # Restart service
systemctl reload service             # Reload config without restart
systemctl enable service             # Enable at boot
systemctl disable service            # Disable at boot

# Service information
systemctl list-units --type=service  # List all services
systemctl list-unit-files           # List all unit files
systemctl cat service               # Show service file contents
journalctl -u service               # View service logs
journalctl -u service -f            # Follow service logs
journalctl -u service --since today # Today's logs
```

### Package Management

#### Debian/Ubuntu (APT)
```bash
# Update and upgrade
apt update                           # Update package lists
apt upgrade                          # Upgrade all packages
apt full-upgrade                     # Upgrade with dependency changes

# Package operations
apt install package                  # Install package
apt remove package                   # Remove package (keep config)
apt purge package                    # Remove package and config
apt autoremove                       # Remove unnecessary dependencies

# Package information
apt search keyword                   # Search for packages
apt show package                     # Show package details
apt list --installed                 # List installed packages
dpkg -l                             # List all packages
dpkg -S /path/to/file               # Find package owning file
```

#### RHEL/CentOS/Fedora (DNF/YUM)
```bash
# Update and upgrade
dnf update                           # Update all packages
dnf upgrade                          # Upgrade all packages

# Package operations
dnf install package                  # Install package
dnf remove package                   # Remove package
dnf autoremove                       # Remove unnecessary dependencies

# Package information
dnf search keyword                   # Search for packages
dnf info package                     # Show package details
dnf list installed                   # List installed packages
rpm -qa                             # List all packages
rpm -qf /path/to/file               # Find package owning file
```

### User Management
```bash
# User operations
useradd username                     # Create user (basic)
useradd -m -s /bin/bash username    # Create with home and shell
passwd username                      # Set/change password
userdel username                     # Delete user
userdel -r username                  # Delete user and home directory
usermod -aG group username          # Add user to group
id username                          # Show user info

# Group operations
groupadd groupname                   # Create group
groupdel groupname                   # Delete group
groups username                      # Show user's groups
cat /etc/passwd                      # List all users
cat /etc/group                       # List all groups

# Switching users
su - username                        # Switch user (login shell)
sudo command                         # Run command as root
sudo -u user command                 # Run command as specific user
sudo -i                             # Interactive root shell
```

### Networking
```bash
# Network configuration
ip addr show                         # Show IP addresses
ip link show                         # Show network interfaces
ip route show                        # Show routing table
ip addr add 192.168.1.10/24 dev eth0 # Add IP address

# Legacy commands (still widely used)
ifconfig                            # Show network interfaces
route -n                            # Show routing table
netstat -tuln                       # Show listening ports

# DNS
dig example.com                      # DNS lookup
nslookup example.com                # DNS lookup (interactive)
host example.com                     # Simple DNS lookup
cat /etc/resolv.conf                # View DNS servers

# Connectivity testing
ping -c 4 example.com               # Ping 4 times
traceroute example.com              # Trace route to host
mtr example.com                     # Combined ping and traceroute

# Network connections
ss -tuln                            # Show listening sockets (modern)
ss -tunp                            # Show all connections with processes
netstat -tuln                       # Legacy version
lsof -i :80                         # Show what's using port 80

# Download files
wget https://example.com/file       # Download file
curl -O https://example.com/file    # Download file (save with same name)
curl https://api.example.com        # Make HTTP request
```

### Disk Management
```bash
# Disk usage
df -h                               # Show disk space (human-readable)
du -sh /path                        # Show directory size
du -h --max-depth=1 /path          # Show sizes of subdirectories

# Mounting
mount                               # Show mounted filesystems
mount /dev/sdb1 /mnt               # Mount filesystem
umount /mnt                         # Unmount filesystem
cat /etc/fstab                      # View filesystem table

# Partition information
lsblk                              # List block devices
fdisk -l                           # List partitions
blkid                              # Show block device attributes
```

### Text Processing
```bash
# Search and filter
grep pattern file                   # Search for pattern
grep -r pattern /path              # Recursive search
grep -i pattern file               # Case-insensitive search
grep -v pattern file               # Invert match (exclude)

# Text manipulation
cat file1 file2 > combined         # Concatenate files
sort file                          # Sort lines
uniq                               # Remove duplicates (requires sorted input)
sort file | uniq                   # Sort and remove duplicates
cut -d: -f1 /etc/passwd           # Extract first field (colon-delimited)
awk '{print $1}' file             # Print first column
sed 's/old/new/g' file            # Replace text
tr 'a-z' 'A-Z' < file             # Translate characters (uppercase)

# Counting
wc -l file                         # Count lines
wc -w file                         # Count words
wc -c file                         # Count bytes
```

### Archiving & Compression
```bash
# tar (tape archive)
tar -czf archive.tar.gz /path      # Create gzip compressed archive
tar -xzf archive.tar.gz            # Extract gzip archive
tar -cjf archive.tar.bz2 /path     # Create bzip2 compressed archive
tar -xjf archive.tar.bz2           # Extract bzip2 archive
tar -tzf archive.tar.gz            # List contents without extracting

# zip
zip -r archive.zip /path           # Create zip archive
unzip archive.zip                  # Extract zip archive
unzip -l archive.zip               # List contents

# Individual compression
gzip file                          # Compress file (creates file.gz)
gunzip file.gz                     # Decompress file
bzip2 file                         # Better compression
bunzip2 file.bz2                   # Decompress bzip2
```

### System Information
```bash
# System details
uname -a                           # All system information
hostname                           # Show hostname
uptime                             # System uptime and load
cat /etc/os-release               # OS information
lscpu                              # CPU information
free -h                            # Memory usage (human-readable)
lsb_release -a                     # Distribution information

# Hardware information
lshw                               # List all hardware
lspci                              # List PCI devices
lsusb                              # List USB devices
dmidecode                          # DMI/SMBIOS information
```

### Logs & Monitoring
```bash
# System logs (systemd)
journalctl                         # All system logs
journalctl -f                      # Follow logs (live)
journalctl -u service              # Service-specific logs
journalctl --since "1 hour ago"    # Recent logs
journalctl -p err                  # Error level and above

# Traditional logs
tail -f /var/log/syslog           # Follow system log
tail -f /var/log/auth.log         # Follow authentication log
less /var/log/messages            # View system messages
dmesg                              # Kernel ring buffer
dmesg | tail                       # Recent kernel messages
```

### Bash Scripting Essentials
```bash
#!/bin/bash
# Shebang - specifies interpreter

# Variables
NAME="value"                       # No spaces around =
echo $NAME                         # Access variable
echo ${NAME}                       # Preferred syntax

# Command substitution
NOW=$(date)                        # Modern syntax
NOW=`date`                         # Legacy syntax

# Conditionals
if [ -f /path/file ]; then
    echo "File exists"
elif [ -d /path/dir ]; then
    echo "Directory exists"
else
    echo "Not found"
fi

# Loops
for i in {1..5}; do
    echo "Number $i"
done

for file in *.txt; do
    echo "Processing $file"
done

while [ condition ]; do
    # commands
done

# Functions
function my_function() {
    echo "Argument: $1"
    return 0
}
my_function "test"

# Exit codes
command
if [ $? -eq 0 ]; then
    echo "Success"
else
    echo "Failed"
fi

# Error handling
set -e                             # Exit on error
set -u                             # Exit on undefined variable
set -o pipefail                    # Exit if any command in pipe fails

# Redirection
command > file                     # Redirect stdout (overwrite)
command >> file                    # Redirect stdout (append)
command 2> file                    # Redirect stderr
command &> file                    # Redirect both stdout and stderr
command1 | command2                # Pipe output to another command
```

## Common Pitfalls & Solutions

### 1. Permission Denied Errors
```bash
# Problem: Cannot execute script
# Solution: Add execute permission
chmod +x script.sh

# Problem: Cannot modify file
# Solution: Check ownership and permissions
ls -l file
sudo chown $USER:$USER file
```

### 2. Disk Space Issues
```bash
# Find large files
find / -type f -size +100M -exec ls -lh {} \;

# Clean package cache
apt clean                          # Debian/Ubuntu
dnf clean all                      # RHEL/CentOS/Fedora

# Remove old logs
journalctl --vacuum-time=7d       # Keep only 7 days of logs
```

### 3. Process Won't Stop
```bash
# Find process
ps aux | grep process_name

# Try graceful kill first
kill PID

# If it doesn't stop, force kill
kill -9 PID
```

### 4. Cannot Connect via SSH
```bash
# Check if SSH service is running
systemctl status sshd

# Check firewall
iptables -L
ufw status                         # Ubuntu firewall

# Check SSH config
cat /etc/ssh/sshd_config

# View SSH logs
journalctl -u sshd -f
```

## Best Practices

1. **Always use absolute paths in scripts** for reliability
2. **Quote variables** to handle spaces: `"$VAR"` not `$VAR`
3. **Check command success** before proceeding: `command && next_command`
4. **Use meaningful names** for scripts and variables
5. **Add comments** to explain complex logic
6. **Test in non-production** before deploying changes
7. **Make backups** before modifying system files: `cp file file.bak`
8. **Use version control** for scripts and configurations
9. **Avoid using root** unless necessary; prefer `sudo`
10. **Monitor logs** when making system changes

## Pro Tips

- **Use tab completion** to speed up commands and avoid typos
- **Use history** to recall previous commands: `Ctrl+R` for search
- **Use aliases** for common commands: `alias ll='ls -lah'`
- **Learn keyboard shortcuts**: `Ctrl+A` (start), `Ctrl+E` (end), `Ctrl+U` (clear line)
- **Use `man` pages** for command documentation: `man command`
- **Check syntax** before running destructive commands
- **Use `screen` or `tmux`** for persistent terminal sessions
- **Keep a personal cheatsheet** of commands you use frequently
