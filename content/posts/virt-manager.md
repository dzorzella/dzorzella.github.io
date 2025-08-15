---
date: '2025-07-31T15:58:05+02:00'
draft: false
title: 'Linux Virtualization'
author: Zorzella Davide

tags: ["virtualization", "qemu", "kvm", "virt-manager"]
categories: ["virtualization"]

cover:
  image: "qemu-logo.svg"
  alt: "qemu logo"
  caption: "qemu logo"
---

A few months ago, I made the decision to fully transition from a Windows-based environment to a Linux desktop setup. This shift marked the culmination of a long and gradual journey that began with running Linux in a VirtualBox environment. From there, I moved on to a dual-boot configuration, and eventually, I reached the point where I completely removed Windows from my system and installed Ubuntu as my sole operating system.

One of the most compelling reasons for this switch was the remarkable performance Linux offers, especially when compared to the sluggish and resource-heavy nature of Windows. However, despite the efficiency of my new setup, I still needed the ability to run additional Linux virtual machines. These were essential for my penetration testing learning path and for conducting low-budget experiments, particularly since I didn’t have access to dedicated hardware for each use case.

To address this need, I decided to explore QEMU in combination with Virtual Machine Manager (using KVM). I found this solution to be exceptionally lightweight, stable, and straightforward to configure—making it an ideal choice for my virtualization needs on a Linux host.


# Install qemu and virtual machine manager

Before installing anything, we proceed with the classic system packages upgrade:

```bash
sudo apt update && sudo apt upgrade
```

After that, we can now install qemu and virtual machine manager packages using:

```bash
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager
```

Once installed, we have to add our user to the `libvirt` and `kvm` groups in order to gain the required privileges:

```bash
sudo adduser $USER libvirt
sudo adduser $USER kvm
```

To make this changes take effect, we should logout and login again.

## libvirtd service

The libvirtd service should be now running 

```bash
sudo systemctl status libvirtd
```

If not, we have to start and enable it

```bash
sudo systemctl start libvirtd
sudo systemctl enable libvirtd
```

# Mount external drive on startup

Mount external drive with kali-linux-2025.1c-qemu-amd64.qcow2 file

/home/$USER/startup.sh

```bash
mkdir /media/davide/DATA
mount -t ntfs /dev/sdb1 /media/davide/DATA -o umask=0000,gid=1000,uid=1000
```

make it root file

```bash
sudo chown root:root startup.sh
sudo chmod 700 startup.sh
```

check ownership and permission

```
-rwx------ 1 root root 100 lug 30 18:12 startup.sh
```

Execute on every startup

```bash
sudo crontab -e
```

```
@reboot /home/davide/startup.sh
```