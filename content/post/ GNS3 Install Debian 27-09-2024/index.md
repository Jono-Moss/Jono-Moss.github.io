+++
author = "Jonathan Moss"
title = 'How To Install GNS3 On Linux - Debian based'
date = 2024-09-27
description = "In this video we go over how to install GNS3 on linux - Debian 12"
tags = [
    "Software",
    "Home Lab",
    "Networking",
]
categories = [
    "Software",
    "Home Lab",
    "Networking",
]
image = "splash.png"
draft = false
+++

## Introduction

We will follow most of the steps in the official Installation guide on GNS3's website, however 
during the installation and initial set up of GNS3. I ran into issues and errors, I will show you all the steps I went through to get GNS3 installed and running. 

## Installation

The official installation guide for Linux can be found using the link below:
https://docs.gns3.com/docs/getting-started/installation/linux/

1. Refresh apt:
```bash
sudo apt update
```

2. Install python and the required emulation & gui packages. When following the original guide, "dynamips" is not found. So in order to continue with the installation, we will remove
it from this step and install it manually in the next step. 
```bash
sudo apt install python3 python3-pip pipx python3-pyqt5 python3-pyqt5.qtwebsockets python3-pyqt5.qtsvg qemu-kvm qemu-utils libvirt-clients libvirt-daemon-system virtinst  software-properties-common ca-certificates curl gnupg2 
```

3. We will now install "dynamips". First we need to download the dynamips package, then install it.

```bash
wget http://ftp.us.debian.org/debian/pool/non-free/d/dynamips/dynamips_0.2.14-1_amd64.deb
```
and then
```bash
sudo dpkg -i dynamips_0.2.14-1_amd64.deb
```

4. Use pipx to install gns3:

```bash
pipx install gns3-server
```
```bash
pipx install gns3-gui
```

We then add the "bin" directory to the "path" for the terminal to be able to find and run GNS3
```bash
pipx ensurepath
```

5. To launch the GUI, we will need to prepare the virtual environment. Inject the GNS server and QT elements:
```bash
pipx inject gns3-gui gns3-server PyQt5
```

6. Close and then reopen your terminal. You can then type "gns3" in the terminal to start it.
```bash
gns3
```

## Fixing Issues and Errors

1. The first issue I ran into is that I would get an "libvirt Not Installed / virbr0 Missing" error when trying to add a "cloud" or "NAT" node. To fix this issue run the following command:

```bash
sudo virsh net-start default
```

2. The next issue I ran into was GNS3 throwing out an "No VPCS Path" error. We will need to do the following steps to fix it.

    2.1  Install the library.
    ```bash
    sudo apt install vpcs
    ```
    After doing this, GNS3 would complain that there is a "Mismatch in the VPCS versions".
    In order to fix this, we now need to build the latest version of VPCS ourselves.

    2.2 First we need to download the source code from Github:
    https://github.com/GNS3/vpcs/releases

    Once downloaded, extract the ".zip" file and cd into the "src" folder
    ```bash
    cd src
    ```

    then run the following command to build the VPCS binary
    ```bash
    ./mk.sh
    ```

    Once it has completed the build. We will remove the old version of VPCS.
    ```bash
    sudo rm /usr/bin/vpcs
    ```

    Then copy the new version to "/usr/bin/". So for example:
    ```bash
    sudo cp /home/youtube/Downloads/vpcs-0.8.3/src/vpcs /usr/bin/vpcs
    ```

That should be all the issues fixed. You can then run GNS3
```bash
gns3
```