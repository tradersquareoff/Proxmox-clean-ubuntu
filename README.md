# Ubuntu 22.04 Server Setup Guide

This guide outlines the basic steps for setting up an Ubuntu 22.04 server with essential configurations for SSH access, root account activation, and security enhancements.

## 1. Install Ubuntu Server

- Download the Ubuntu 22.04 server ISO from the official Ubuntu website.
- Create a bootable USB drive or prepare a virtual machine if installing in a virtualized environment.
- Follow the installation prompts to select your server's hostname, partition disks, and create a user account with `sudo` privileges.

## 2. Activate SSH

Ensure SSH is installed and running:

```bash
sudo apt-get update
sudo apt-get install openssh-server
```

Edit the SSH configuration file:

```bash
sudo nano /etc/ssh/sshd_config
```

Make sure `PermitRootLogin` is set to `yes` or `prohibit-password` and the SSH port is correct (default is 22).

Restart the SSH service to apply changes:

```bash
systemctl restart sshd
```

## 3. Activate Root

Set or change the root password:

```bash
sudo passwd root
```

## 4. Send SSH Key to Server

Copy your public SSH key:

```bash
ssh-copy-id your_username@server_address
```

Alternatively:

```bash
cat ~/.ssh/id_rsa.pub | ssh your_username@server_address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

## 5. Configure SSH for Key-Based Authentication

Edit the SSH configuration to enforce key-based authentication:

```bash
sudo nano /etc/ssh/sshd_config
```

Change `PermitRootLogin` to `prohibit-password`. Restart the SSH service:

```bash
sudo systemctl restart sshd
```

## 6. Install Fail2Ban

Install Fail2Ban for additional security:

```bash
sudo apt-get update
sudo apt-get install fail2ban
```

Ensure Fail2Ban starts automatically:

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

## 7. Install QEMU Guest Agent (For Proxmox Only)

```bash
sudo apt-get install qemu-guest-agent
sudo systemctl start qemu-guest-agent
sudo systemctl enable qemu-guest-agent
```

This tutorial provides a basic framework for setting up an Ubuntu 22.04 server. Remember to secure your server according to your organizational security policies and requirements.
