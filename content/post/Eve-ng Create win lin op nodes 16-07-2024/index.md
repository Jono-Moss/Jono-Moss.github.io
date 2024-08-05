+++
author = "Jonathan Moss"
title = 'How to create a Windows, Linux and OpnSense node in EVE-NG'
date = 2024-07-16
description = "In this guide, I go over how to create a Windows, Linux and OpnSense node in EVE-NG"
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

## Nodes

In this guide we will create a Windows 10, OpnSense and a Debian 12 node.  
The official notes can be found on EVE-NG's website:  
https://www.eve-ng.net/index.php/documentation/howtos/

There are guides on how to add and create all available nodes on EVE-NG. So it is recommended to have a look at them in order for you to add the nodes you require.

### Node VM Creations

#### Download the ISO's

Download the desired ISO's. In my case I have downloaded:  
OpnSense 24  
Windows 10 (64 bit)  
Debian 12 (net Installer)

#### Create the folders on EVE-NG

SSH into the EVE-NG machine and create a folder in the:  
`bash /opt/unetlab/addons/qemu/`  
folder  
For example I created the following:

```bash
mkdir /opt/unetlab/addons/qemu/win-10x64Pro/
mkdir /opt/unetlab/addons/qemu/linux-debian-12/
mkdir /opt/unetlab/addons/qemu/opnsense-24/
```

#### Copy the ISO's to the folders

Next I will use `SCP` to copy the ISO's. from my downloads folder into the desired folder we have created for the VM's.  
It is important that we name the ISO `cdrom.iso` when we add it to the folder. Here are my `SCP` commands

OpnSense:

```bash
scp /Users/jono/Downloads/OPNsense-24.1-dvd-amd64.iso root@10.0.0.9:/opt/unetlab/addons/qemu/opnsense-24/cdrom.iso
```

Debian 12:

```bash
scp /Users/jono/Downloads/debian-12.6.0-amd64-netinst.iso root@10.0.0.9:/opt/unetlab/addons/qemu/linux-debian-12/cdrom.iso
```

Windows 10:

```bash
scp /Users/jono/Downloads/Win10_22H2_EnglishInternational_x64v1.iso root@10.0.0.9:/opt/unetlab/addons/qemu/win-10x64Pro/cdrom.iso
```

#### Create The Virtual Disks (HDD's) for the VM's

Next we need to create the virtual disks for each VM / Node to use.  
SSH into the EVE-NG again and do the following:

OpnSense:

```bash
cd /opt/unetlab/addons/qemu/opnsense-24/
/opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 10G
```

Debian 12:

```bash
cd /opt/unetlab/addons/qemu/linux-debian-12/
/opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 30G
```

Windows 10:

```bash
cd /opt/unetlab/addons/qemu/win-10x64Pro/
/opt/qemu/bin/qemu-img create -f qcow2 virtioa.qcow2 60G
```

#### Installing the OSes on the VMs / Nodes

1.  Log In to EVE-NG Using the "HTML 5" session.
2.  Create a new lab and add the newly created nodes.
3.  Connect the node to your home LAN cloud/internet in order for it to be able to get updates and fully install from the internet. However, do not add the Windows 10 node to the internet, this will just quickly allow us to bypass the "Online Microsoft Account" link stage.
4.  start each node and complete the installation of. the OSes.
5.  Once completed, Shutdown and stop each node.

#### Commit VM's

1.  Check lab ID number on EVE side bar “Lab details” and we need to also get the "Node Number". number of each node which can be seen in the "Nodes" List
    
2.  SSH into the EVE-NG machine and `cd` into the labs folder.
    

```bash
cd /opt/unetlab/tmp/0/the-lab-id-here/
```

3.  We will go into the nodes one by one and convert the temp VM to be the "main" vm for all future nodes of that type. For example

```bash
cd /opt/unetlab/tmp/0/the-lab-id-here/0/
qemu-img commit virtioa.qcow2

cd /opt/unetlab/tmp/0/the-lab-id-here/1/
qemu-img commit virtioa.qcow2

cd /opt/unetlab/tmp/0/the-lab-id-here/2/
qemu-img commit virtioa.qcow2
```

#### Removes ISO's from the machine

Now we need to `cd` into each the VM folders again and remove the ISO images.

OpnSense:

```bash
cd /opt/unetlab/addons/qemu/opnsense-24/
rm -f cdrom.iso
```

Debian 12:

```bash
cd /opt/unetlab/addons/qemu/linux-debian-12/
rm -f cdrom.iso
```

Windows 10:

```bash
cd /opt/unetlab/addons/qemu/win-10x64Pro/
rm -f cdrom.iso
```

#### Fix file Permissions

lastly we need to run the following to fix any permission issues

```bash
/opt/unetlab/wrappers/unl_wrapper -a fixpermissions
```

#### Optional - Compress the VM's HDD's

1.  You can `cd` into each VM's folder again and sparsify the virtual disk to compress the them, for example to compress the windows 10 VM we will do the following:

```bash
cd /opt/unetlab/addons/qemu/win-10x64Pro/
virt-sparsify  --compress virtioa.qcow2 compressedvirtioa.qcow2
```

This will take some time and another compressed image will be created in the same image directory

2.  Rename the original image to virtioa-o.qcow2:

```bash 
mv virtioa.qcow2 virtioa-o.qcow2
```

3.  Rename the compressed image name to virtioa.qcow2:
```bash 
mv compressedvirtioa.qcow2 virtioa.qcow2
```
4.  now you can test your new compressed image on a lab, just wipe the node and start it.

If the compressed node works fine, you can delete your original source image:

```bash
rm -f. virtioa-o.qcow2
```

**Please note**
I tried to compress windows 10 and when using it, it would not boot correctly. I attempted this twice and had issues. So I reverted to use the original virtual disk.

## WebUI Access Method

### To setup OpnSense web UI from wan

1.  in shell on opnsense: `pfctl -d`
2.  You ca now access GUI via WAN IP
3.  disable the "Block bogon" traffic:
    1.  interfaces->WAN "Untick" both "block private networks" and block "Bogon Networks"
4.  I would also go to Firewall > Settings > Advanced and check "Disable anti-lockout".
5.  Firewall --> Settings--> Advanced "Disable reply-to on WAN rules"
6.  create the following Firewall rule.

```
Firewall > Rules > WAN

Click on the "+" to add a rule. Add this rule:

Action: allow
Source: any
Destination: This Firewall
Destination port: 443
```