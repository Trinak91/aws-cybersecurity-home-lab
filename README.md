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
