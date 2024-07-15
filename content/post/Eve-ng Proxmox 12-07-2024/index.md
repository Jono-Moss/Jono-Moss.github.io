+++
author = "Jonathan Moss"
title = 'How to install EVE-NG on Proxmox'
date = 2024-07-15
description = "In this video we show you how to install EVE-NG on Proxmox'."
tags = [
    "Software",
    "Home Lab",
    "Networking",
    "Virtual Machine",
]
categories = [
    "Software",
    "Home Lab",
    "Networking",
    "Virtual Machine",
]
series = ["EVE-NG"]
image = "splash.jpg"
draft = false
+++

{{< youtube IEMYJscrykI >}}

## Introduction
Hi there, Jono here, on a regular basis, I need to test "Use Cases", Software configurations and network setups for customers. I also wanted to start working on guides showing more advanced setups that would require multiple servers, networking equipment and cable which can be problematic, both in terms of cost and physical space. 
So I was thinking about how I could use my single home lab server to better achieve this "juggling" of tasks and that was when it hit me.

Why not use Eve-ng or GNS3?

Eve-ng or "emulated virtual environments, next generation" and GNS3 or "Graphical Network Simulator - 3" are in a sense "hypervisors" which allow you to create virtual machines or containers, which in Eve-ng are called "nodes" and in "GNS3" are called "Appliances". These. are then used in a "drag and drop" fashion where you quickly and easily create virtual networks which can then be used. in real world deployment.

In the past, I never really owned a device that was powerful enough to run these tools, but. now-a-days, hardware. has become so powerful, you can run these tools on relatively cheap laptops and PC's.

Since it has been so long since I have tried them, I will be giving them both a try again. In this guide I will be using Eve-ng's community edition, and will do a few "labs" in it to give it a fair test. In future guides I will move over to GNS3 and after giving that a good test, I will make a video on my experiences.

Now lets move onto the meat of this guide,  and that is how to install Eve-ng on a virtual machine in Proxmox.

## Installation

### Download the ISO
Download the community edition from:
https://www.eve-ng.net/index.php/download/#DL-COMM
and upload it to Proxmox

### Enable Nested Virtualisation
Nested virtualisation allows you to run a hypervisor within a virtual machine, which is how these tools work, we need to check that is is enabled on Proxmox. If I remember correctly,  it used to be disabled by default, however, will all my new Proxmox installations, It appears that it is now enabled by default

Using SSH or using Proxmox's built in shell:
// For Intel
```
cat /sys/module/kvm_intel/parameters/nested
```

// For AMD
```
cat /sys/module/kvm_amd/parameters/nested
```

It will either show a "Y" or "N". "Y" to indicate if it is enabled or "N" if not.

To enable nested virtualisation you just have to do a few simple things, I’d suggest to follow the official article, 
https://pve.proxmox.com/wiki/Nested_Virtualization
incase this changes in the future, but as of today, you can use this quick rundown to get up and running:

// For Intel
```bash
echo "options kvm-intel nested=Y" > /etc/modprobe.d/kvm-intel.conf
```

// For AMD
```bash
echo "options kvm-amd nested=1" > /etc/modprobe.d/kvm-amd.conf
```

Reboot or reload the kernel module

// For Intel
```bash
modprobe -r kvm_intel
modprobe kvm_intel
```

// For AMD
```bash
modprobe -r kvm_amd
modprobe kvm_amd
```

It is recommend to do a full reboot.
After the reboot, check if the module is enabled.

### Create the Virtual Machine
Create a Virtual machine with the following:
100GB Disk Space (More the merrier)
+- 4GB RAM (More the merrier)
CPU set to "Host" (This is a must)
Make sure to attach the Eve-ng ISO
Attach it to a network that has internet access (This is a must)
the rest is up to you.

Just a quick notice, that in the video I set the HDD to use "VirtIO" in stead of "SCSI" as a saw a few people mention that they where getting better results. However I have not tested this claim myself. It is up to you to decide what you would like to use.

### Eve-ng installation
Eve-ng uses "cloud-init" to do a lot of the installation. So depending o the speed of your internet, this process can take a long time.
once it is. completed, you will be presented with the configuration phase where you will be guided through configuring your instance.

In this case I have set the IP to be 10.0.0.9

### Access the Web UI
once the installation is completed. You can goto the the instance IP in your favourite browser to access the Web UI.
The default login details are: 
admin
eve

### Reconfigure your Instance
If. you have made a mistake during the "Configuration" stage. You can do the following to run it again.

SSH into it or directly via the console. 

Delete the "Configured" file
```
rm -f /opt/ovf/.configured
```

and then 
```
su -
```
This will rerun the configuration stage.

## Conclusion
You will now have an Eve-ng Vm up and running, almost ready to use. All that is left is to create "nodes" that we can use.
In the next guide, we will create an OpnSense "node" and I will show you the method I used to get access to the OpnSense WebUI of that "node" in order to use and configure it.