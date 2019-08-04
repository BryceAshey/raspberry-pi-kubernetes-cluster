# Raspberry Pi Setup

### update kb to us
sudo nano /etc/default/keyboard

## Hardening
# https://www.raspberrypi.org/documentation/configuration/security.md

# Create new user
sudo adduser kadmin
sudo adduser kadmin sudo

# Test then remove pi
sudo deluser pi

# Make sudo require a password on kadmin (change 'pi' to 'kadmin')
sudo nano /etc/sudoers.d/010_pi-nopasswd	

# Install fail2ban
sudo apt install fail2ban

# copy fail2ban config to local
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# add the following under sshd
sudo nano /etc/fail2ban/jail.local

[ssh]
enabled  = true
port     = ssh
filter   = sshd
logpath  = /var/log/auth.log
maxretry = 3
findtime = 1800
bantime = 1800

# restart fail2ban
sudo systemctl enable fail2ban
sudo systemctl restart fail2ban


# Basic configuration

# disable swap
sudo dphys-swapfile swapoff && \
  sudo dphys-swapfile uninstall && \
  sudo update-rc.d dphys-swapfile remove

# update boot
sudo nano /boot/cmdline.txt
# add to end
 cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory

# disable wifi
echo "dtoverlay=pi3-disable-wifi" | sudo tee -a /boot/config.txt

# setup static ip
sudo nano /etc/dhcpcd.conf
sudo systemctl daemon-reload
sudo service dhcpcd restart

	domain iot.shroudedmountain.com
	nameserver 192.80.1.82
	nameserver 192.80.1.90


# get latest
sudo apt-get update
sudo apt-get upgrade

# remove pulse audio
sudo apt-get -y purge "pulseaudio*"

# Docker & Kubernetes Setup

# install Docker
curl -sL get.docker.com | sed 's/9)/10)/' | sh
#curl -sSL get.docker.com | sh && \

# create docker group
sudo usermod kadmin -aG docker
newgrp docker

# install kubernetes
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubeadm

# Configure Kubernetes

sudo kubeadm config images pull -v3

sudo kubeadm init --token-ttl=0 --pod-network-cidr=172.16.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Make note of join command

# configure Flannel
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
sudo sysctl net.bridge.bridge-nf-call-iptables=1

# Configure K8 dashboard
#do this: https://github.com/kubernetes/dashboard/wiki/Access-control#admin-privileges

