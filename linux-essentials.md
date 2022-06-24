# General configuration notes

## Using SWAP file

Configuration sequence using bash:

```bash
fallocate -l 1G /swapfile
# Or use dd if fallocate is N/A
# dd if=/dev/zero of=/swapfile bs=1024 count=1048576
chown root:root /swapfile
chmod 0600 /swapfile
mkswap /swapfile
swapon /swapfile
# Add swap to fstab
# /swapfile swap swap defaults 0 0
# Verify
swapon --show
# free -h
```

Swappiness is a Linux kernel property that defines how often the system will use the swap space. Swappiness can have a value between 0 and 100. A low value will make the kernel to try to avoid swapping whenever possible while a higher value will make the kernel to use the swap space more aggressively.

```bash
cat /proc/sys/vm/swappiness
# Update file /etc/sysctl.conf
# vm.swappiness=10
```

## Create new user

```bash
useradd username
# Create with home directory -m or --create-home
# useradd --create-home username
# useradd -m -d /opt/username username
useradd -m username
passwd username
# Add user to group users
useradd -g users username
id -gn username
```

The following command creates a new user named username with primary group users and secondary groups wheel and docker.

```bash
useradd -g users -G wheel,developers username
id username
```

## Set up a Firewall with ufw

UFW, or Uncomplicated Firewall, is an interface to iptables that is geared towards simplifying the process of configuring a firewall. While iptables is a solid and flexible tool, it can be difficult for beginners to learn how to use it to properly configure a firewall.

```bash
apt install ufw
# nano /etc/default/ufw
# IPV6=yes

# Set defaults
ufw default deny incoming
ufw default allow outgoing
ufw enable
ufw disable
ufw reload

# Allow connections
# ufw allow 22
ufw allow ssh
# Allow ranges and specific IP addresses and subnets
ufw allow 6000:6007/tcp
ufw allow 6000:6007/udp
ufw allow from 203.0.113.4
ufw allow from 203.0.113.4 to any port 22
ufw allow from 203.0.113.0/24
ufw allow from 203.0.113.0/24 to any port 22
ufw allow proto tcp from any to 10.8.0.1 port 22
ufw allow proto tcp from 10.8.0.2 to 10.8.0.1 port 22
ufw allow in on eth0 to any port 80
ufw allow in on eth1 to any port 3306
ufw allow from 192.168.1.0/24 to any port 3306
ufw allow from 192.168.1.0/24 to any port 5432
# Rules with comments
ufw allow https/tcp comment 'Open port Apache port 443'


# Deny connections
ufw deny http
ufw deny from 203.0.113.4
ufw deny from 123.45.67.89/24
ufw deny from 1.2.3.4 to any port 22 proto tcp

# Reject connections (Let user know they are blocked by the firewall)
ufw reject in smtp
ufw reject out smtp
ufw reject 1194 comment 'No more vpn traffic'
ufw reject 23 comment 'Unencrypted port not allowed'
# Example response: telnet: Unable to connect to remote host: Connection refused

# Delete rules
ufw status numbered
# If we decide that we want to delete rule 2
ufw delete 2
ufw delete allow http
ufw delete allow 80
# Reset all configured UFW rules
ufw reset

# Status
ufw status verbose
# Check rules before starting firewall
ufw show added
ufw show listening
ufw show builtins
ufw show before-rules
ufw show user-rules
ufw show after-rules
ufw show logging-rules
```

## Tuning Linux File System on Dynamic VHDX Files

Some Linux file systems may consume significant amounts of real disk space even when the file system is mostly empty. To reduce the amount of real disk space usage of dynamic VHDX files, consider the following recommendations:

```powershell
New-VHD -Path C:\MyVHDs\test.vhdx -SizeBytes 127GB -Dynamic -BlockSizeBytes 1MB
```

```bash
mkfs.ext4 -G 4096 /dev/sdX1
```

> After resizing a VHD or VHDX, administrators should use a utility like fdisk or parted to update the partition, volume, and file system structures to reflect the change in the size of the disk

Shrinking or expanding the size of a VHD or VHDX that has a GUID Partition Table (GPT) will cause a warning when a partition management tool is used to check the partition layout, and the administrator will be warned to fix the first and secondary GPT headers.

> This manual step is safe to perform without data loss.

The VHD format was the only virtual hard disk format that was supported by Hyper-V in past releases. Introduced in Windows Server 2012, the VHD format has been modified to allow better alignment, which results in significantly better performance on new large sector disks.

>Any new VHD that is created on a Windows Server 2012 or newer has the optimal 4 KB alignment.

This aligned format is completely compatible with previous Windows Server operating systems. However, the alignment property will be broken for new allocations from parsers that are not 4 KB alignment-aware (such as a VHD parser from a previous version of Windows Server or a non-Microsoft parser).

## Configuring right I/O scheduler

The Linux kernel offers two sets of disk I/O schedulers to reorder requests. One set is for the older **blk** subsystem and one set is for the newer **blk-mq** subsystem. In either case, with today’s solid state disks it is recommended to use a scheduler that passes the scheduling decisions to the underlying Hyper-V hypervisor.

* For Linux kernels using the **blk** subsystem, this is the **noop** scheduler.
* For Linux kernels using the **blk-mq** subsystem, this is the **none** scheduler.
* Note: Single-queue schedulers (noop, deadline, cfq) were removed from kernel since Linux 5.0

```bash
# List the available schedulers for a device and the active scheduler (in brackets)
cat /sys/block/sda/queue/scheduler

# To list the available schedulers for all devices
grep "" /sys/block/*/queue/scheduler

# To change the active I/O scheduler to bfq for device sda, use
echo bfq > /sys/block/sda/queue/scheduler
```
