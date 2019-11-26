# Node Setup

### Raspberry PI Image
Download the [Raspbian Buster Lite](https://downloads.raspberrypi.org/raspbian_lite/images/raspbian_lite-2019-07-12/2019-07-10-raspbian-buster-lite.zip) image onto your SD card and boot your PIs up.

### Via raspi-config:
- update hostname
- expand filesystem
- enable ssh
- update timezone as appropriate

### General Setup
We highly recommend reading Raspberry's [guidance](https://www.raspberrypi.org/documentation/configuration/security.md) on security best practices.


If in the US update your keyboard to "US"
```
> sudo nano /etc/default/keyboard
```

Create the Kubernetes admin user and add the user to sudo'ers
```
> sudo adduser kadmin
> sudo adduser kadmin sudo
```

Exit and login as kadmin

Delete the default user
```
> sudo deluser pi
```

Require sudo password on kadim. Change "pi" to "kadmin" in the file below
```
> sudo nano /etc/sudoers.d/010_pi-nopasswd
```

Optionally you may want to install [fail2ban](https://www.fail2ban.org/wiki/index.php/Main_Page) to lock the account after a number of tries...

Make dang sure that swap is turned off. Previously we could simply disable it as below but with Buster it appears that we need to now set swapsize = 0 to truly disable it.
```
> sudo dphys-swapfile swapoff
> sudo dphys-swapfile uninstall
> sudo update-rc.d dphys-swapfile remove

> sudo nano /etc/dphys-swapfile
# find CONF_SWAPSIZE=100 and change it to CONF_SWAPSIZE=0
```

Configure cgroup
```
> sudo nano /boot/cmdline.txt
# add the below line to the end of the existing text. Make sure to put a space (not carriage return) after the last value and the values below:
 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory
```

Disable the WIFI adapter
```
> echo "dtoverlay=pi3-disable-wifi" | sudo tee -a /boot/config.txt
```

Setup iptables to forward packets (required by Kubernetes networking)
```
> sudo iptables -P FORWARD ACCEPT
```

Add the above line to /etc/rc.local so that it gets reset on every boot
```
> sudo nano /etc/rc.local
# Paste the following line above "Exit 0" and make sure the top of the file has "#!/bin/sh -e"
/sbin/iptables -P FORWARD ACCEPT
```

Allow non-local bind (required by HAProxy)
Allow bridge to call iptables
```
> sudo sysctl net.ipv4.ip_nonlocal_bind = 1
> sudo sysctl net.bridge.bridge-nf-call-iptables=1
```

On each node configure your networking with a static IP. Alternatively you could configure DHCP on your router to assign a static IP to each server. Change the IPs in the list below to match your assignments.
```
> sudo nano /etc/hosts

# paste the following values at the end of the file
192.168.200.249 kubernetes
192.168.200.250 kuber04m01
192.168.200.251 kuber04m02
192.168.200.252 kuber04m03
192.168.200.230 kuber04w01
192.168.200.231 kuber04w02
192.168.200.232 kuber04w03
192.168.200.233 kuber04w04
```

Update your distribution
```
> sudo apt-get update
> sudo apt-get upgrade
```

Free up some space
```
> sudo apt-get -y purge "pulseaudio*"
```

Reboot to make sure everything is fresh
```
> sudo reboot now
```

## Install Docker and Kubernetes

### Install Docker

Note the hack here to install 9 instead of 10 on Buster. At the time of this writing there was not yet an ARM64 version of Docker compiled for Buster.
```
> curl -sL get.docker.com | sed 's/9)/10)/' | sh

# Once an ARM64 version is available the standard command (below) should work again
# curl -sSL get.docker.com | sh
```

Add kadmin to the docker group
```
> sudo usermod kadmin -aG docker
```

### Install Kubernetes

```
> curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubeadm
```

On the **master** nodes only; prepull the images for kubernetes
```
> sudo kubeadm config images pull -v3
```

### [Optional] Enable CIFS volumes
```
> sudo apt-get install -y jq
> VOLUME_PLUGIN_DIR="/usr/libexec/kubernetes/kubelet-plugins/volume/exec"
> sudo mkdir -p "$VOLUME_PLUGIN_DIR/fstab~cifs"
> cd "$VOLUME_PLUGIN_DIR/fstab~cifs"
> sudo curl -L -O https://raw.githubusercontent.com/fstab/cifs/master/cifs
> sudo chmod 755 cifs

# Verify the installation
> $VOLUME_PLUGIN_DIR/fstab~cifs/cifs init
```

Reboot again - ran into an issue once or twice when I didn't reboot
```
> sudo reboot now
```
