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
## 8. Install zram, and activate it (50 % of RAM as compressed RAM), check with swapon or zramctl
The disksize you set during zram device creation is basically a virtual disk size, this does not stand for the real RAM usage.
You have to predict the actual RAM usage (compression ratio) for your scenario, or make sure that you never create a zram disk size that is too large. Your current RAM size + 50% should be nearly always fine in practice.
The default configuration of the Linux kernel is unfortunately totally unsuited for zram compression, even when setting vm.swappiness to 100. You need to make your own custom kernel to actually make real use of this handy feature, since Linux purges way too many file caches instead of freeing up memory by swapping the most compressible data much earlier. Ironically, a helpful patch to fix this situation was never accepted.
Using the zram limit echo 1G > /sys/block/zram0/mem_limit will lock up your system once the compressed data reached that threshold. You are better off to limit zram usage with a well-predicted zram disksize, as it seems there is no other alternative for a limit.
```bash
apt-get -uy install zram-config nohang; service zram-config start
# alternatively go with debian-system-zram (also works on Ubuntu)
# also read:
# https://docs.kernel.org/admin-guide/blockdev/zram.html
# https://unix.stackexchange.com/questions/594817/why-does-zram-occupy-much-more-memory-compared-to-its-compressed-value
# https://lore.kernel.org/lkml/20160606194836.3624-2-hannes@cmpxchg.org/
# https://unix.stackexchange.com/questions/32333/what-does-the-vm-swappiness-parameter-really-control
```

## 9. Remove Ubuntu Spyware

```bash
apt-get -uy --purge remove whoopsie kerneloops modemmanager apport apport-symptoms unattended-upgrades gnome-online-accounts switcheroo-control
apt-get -uy --purge remove update-notifier-common apport ipp-usb ubuntu-release-upgrader-gtk update-notifier
```

This tutorial provides a basic framework for setting up an Ubuntu 22.04 server. Remember to secure your server according to your organizational security policies and requirements.

## Install multiple instances of GO with GVM

```bash
sudo apt install bsdmainutils -y
```

```bash
apt-get install bison -y
```

```bash
apt-get install nano -y
```

```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

```bash
source /root/.gvm/scripts/gvm
```


(optionnel)
```bash
nano ~/.bashrc
```

```bash
gvm install go1.4 -B
gvm use go1.4
export GOROOT_BOOTSTRAP=$GOROOT
gvm install go1.17.13
gvm use go1.17.13
export GOROOT_BOOTSTRAP=$GOROOT
gvm install go1.20.14
gvm use go1.20.14
```

```bash
go version
```

```bash
git clone https://github.com/QuilibriumNetwork/ceremonyclient.git
cd ceremonyclient/node
GOEXPERIMENT=arenas go install ./...
ls /root/.gvm/pkgsets/go1.20.14/global/bin
```

```bash
GOEXPERIMENT=arenas go run ./...
```
open a new bash
```bash
ps aux
```

```bash
kill -9 PUID
```

```bash
nano .config/config.yml
```
  `listenGrpcMultiaddr: /ip4/127.0.0.1/tcp/8337`
and under 
  engine:
    `statsMultiaddr: "/dns/stats.quilibrium.com/tcp/443"`

```bash
cd
cd ceremonyclient/node
./poor_mans_cd.sh
```

## Install Docker

1. **Add Docker's official GPG key:**

   ```bash
   sudo apt-get update
   sudo apt-get install ca-certificates curl
   sudo install -m 0755 -d /etc/apt/keyrings
   sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
   sudo chmod a+r /etc/apt/keyrings/docker.asc
   ```

2. **Add the repository to Apt sources:**

   ```bash
   echo \
   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
   $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt-get update
   ```

3. **Install Docker**

   ```bash
   sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
   ```
