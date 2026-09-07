# aws-cybersecurity-home-lab
Hands-on AWS cybersecurity lab demonstrating cloud security, IAM, network security, logging, and incident investigation.

# AWS Cybersecurity Lab

## Overview

This project is a hands-on cybersecurity lab built using Amazon Web Services (AWS) and Ubuntu Linux.

The goal of this project is to develop practical experience with cloud networking, Linux administration, firewall configuration, system hardening, and security monitoring.

## Environment

- Cloud Provider: AWS
- Region: US East (Ohio)
- Operating System: Ubuntu Server 24.04 LTS
- EC2 Instance: Cybersecurity-Lab-Server
- VPC CIDR: 10.0.0.0/16
- Public Subnet: 10.0.0.0/20

## Network Architecture

Internet
   |
Internet Gateway
   |
AWS VPC
10.0.0.0/16
   |
Public Subnet
10.0.0.0/20
   |
Security Group
SSH - TCP 22 - My IP
   |
Ubuntu EC2 Server

## Security Controls

### AWS Security Group

SSH access was restricted to my public IP address rather than allowing SSH access from the entire internet.

### UFW Firewall

UFW was enabled on the Ubuntu server.

Configuration:

- Default incoming traffic: DENY
- Default outgoing traffic: ALLOW
- SSH: TCP/22 allowed

### System Updates

The Ubuntu operating system was updated using:

sudo apt update
sudo apt upgrade

The server required a reboot to activate a new Linux kernel.

The active kernel was verified using:

uname -r

## Network Exposure Assessment

I used the following command to identify listening network services:

sudo ss -tulpn

### Findings

- SSH was listening on TCP port 22.
- DNS resolution services were bound to localhost.
- Chrony was bound to localhost.
- No unnecessary public-facing services were identified.

## Security Principles Practiced

- Least privilege
- Defense in depth
- Network segmentation
- System patching
- Firewall configuration
- Attack surface assessment
- Secure remote administration

## Lessons Learned

This project helped me understand how AWS networking components work together, including VPCs, subnets, route tables, Internet Gateways, and security groups.

I also gained hands-on experience administering an Ubuntu Linux server over SSH and configuring a host-based firewall.

## Future Improvements

- Configure centralized logging
- Implement intrusion detection
- Install and configure Fail2Ban
- Create a private subnet
- Deploy a second server
- Implement additional IAM controls
- Perform controlled security testing
  ## User and Account Assessment

I reviewed currently logged-in users using:

who

The assessment identified one active interactive session belonging to the
ubuntu administrative account.

I also reviewed local accounts using:

sudo getent passwd

The majority of accounts were system/service accounts and used
/usr/sbin/nologin or /bin/false as their login shell, preventing normal
interactive login.

The primary administrative account was the ubuntu account. The root
account exists but was not used for direct SSH authentication.

### Finding

No unexpected interactive user accounts were identified during the review.
Service accounts were generally configured without interactive login shells,
which reduces unnecessary login exposure.

### Sudo Privilege Assessment

I reviewed the administrative privileges assigned to the ubuntu account
using:

bash
sudo -l

## User and Account Assessment
### Sudo Privilege Assessment

The ubuntu account was configured with full sudo privileges, including
passwordless execution of commands as root.

This configuration is common for Ubuntu AWS instances, where SSH key
authentication is used for initial administrative access.

### Security Consideration

Because the ubuntu account can obtain full root privileges, compromise
of this account could result in complete system compromise.

To reduce this risk, SSH access is restricted at the AWS Security Group
level to my IP address, and the Ubuntu UFW firewall is enabled with a
default-deny inbound policy.

Protecting the SSH private key and limiting remote access are therefore
critical security controls.

## SSH Security Assessment

The SSH service was reviewed to evaluate authentication and remote-access security.

### Findings

- Public-key authentication is enabled.
- Password-based SSH authentication is disabled.
- Direct root SSH access is currently configured to allow key-based authentication.

### Security Controls

The server uses SSH key-based authentication rather than passwords. Password authentication is disabled, reducing the risk of password-based brute-force attacks.

SSH access is also restricted through the AWS Security Group to my IP address, while UFW is enabled with a default-deny inbound policy.

### Recommended Hardening

Direct root SSH access should be disabled entirely. Administrative access should instead use the `ubuntu` account with `sudo` privileges.
### SSH Hardening Finding

The initial SSH configuration allowed direct root login using
public-key authentication. The configuration reported:

    permitrootlogin without-password
    pubkeyauthentication yes
    passwordauthentication no

Although password-based SSH authentication was disabled, allowing
direct root access through SSH increases the impact of a compromised
root credential.

### Remediation

To follow the principle of least privilege, direct root SSH access
was disabled by configuring:

    PermitRootLogin no

Administrative tasks will instead be performed through the
non-root `ubuntu` account using `sudo`.

A backup of the original SSH configuration was created before
making the change:

    /etc/ssh/sshd_config.backup
### Verification

After applying the SSH hardening change, the SSH configuration was
validated with `sshd -t` and returned no errors.

The active SSH configuration was then verified with `sshd -T`:

    permitrootlogin no
    pubkeyauthentication yes
    passwordauthentication no

This confirms that direct root SSH access has been disabled while
public-key authentication remains enabled and password-based SSH
authentication remains disabled.
## Linux File Permission Assessment

### /etc/passwd

The permissions of `/etc/passwd` were reviewed using `ls -l`.

    -rw-r--r-- 1 root root /etc/passwd

The file is owned by `root` and is writable only by the root user.
Other users have read-only access, which is expected because Linux
systems require account information in `/etc/passwd` to be readable.

### Security Assessment

No excessive write permissions were identified on `/etc/passwd`.
Restricting write access to root helps prevent unauthorized
modification of system account information.

### /etc/shadow

The permissions of `/etc/shadow` were reviewed using `ls -l`.

    -rw-r----- 1 root shadow /etc/shadow

The file is owned by `root` and assigned to the `shadow` group. Only
the root user and members of the `shadow` group have read access.
Other users have no permissions on the file.

### Security Assessment

The permissions on `/etc/shadow` are appropriately restrictive.
Because this file contains password hashes and authentication-related
information, limiting access helps reduce the risk of unauthorized
access to credential data.

This follows the principle of least privilege and reduces the risk associated with direct remote root access.
### /etc/ssh

The permissions of the SSH configuration directory were reviewed
using `ls -ld`.

    drwxr-xr-x 4 root root /etc/ssh

The directory is owned by `root` and is writable only by the root
user. Other users have read and execute permissions but cannot
modify, create, or delete files within the directory.

### Security Assessment

The `/etc/ssh` directory permissions are appropriately configured.  
### /etc/ssh/sshd_config

The permissions of the SSH server configuration file were reviewed
using `ls -l`.

    -rw-r--r-- 1 root root /etc/ssh/sshd_config

The file is owned by `root` and is writable only by the root user.
Other users have read-only access.

### Security Assessment

The SSH configuration file has appropriate write permissions.
Restricting modification to root helps prevent unauthorized changes
to SSH authentication and access-control settings.
Restricting write access to root helps prevent unauthorized
modification of SSH configuration files and host keys.
