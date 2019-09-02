# Raspberry PI Based Kubernetes Cluster
Designs, instructions, and more for a seven node Raspberry PI Kubernetes cluster.

[**Bill of Materials**](docs/BOM.md)

**Total Build Time:** Expert 1 week / Newbie 4 - 8 weeks

Physical Construction

### [All Node Setup](docs/all-node-setup.md)
Each node requires some basic setup to prep it for use with Kubernetes.

### [Master Node Setup](docs/master-node-setup.md)


### [Gluster Setup](docs/gluster-setup.md)
In this implementation we use [Gluster](https://gluster.org) as the backing filesytem for Kubernetes Persistent Volumes. Gluster provides a fault-tolerant system across commodity and disparate hardware.

### [Kubernetes Setup](docs/kubernetes-setup.md)


## Additional Reading
- [Igor Cicimov's Kubernetes cluster step-by-step](https://icicimov.github.io/blog/kubernetes/Kubernetes-cluster-step-by-step/)
- [Securing Raspbian](https://www.raspberrypi.org/documentation/configuration/security.md)
- [HAProxy Web Server on Raspbian](http://gregtrowbridge.com/setting-up-a-multiple-raspberry-pi-web-server-part-5/)
- [High Availability HAProxy](https://www.digitalocean.com/community/tutorials/how-to-create-a-high-availability-haproxy-setup-with-corosync-pacemaker-and-floating-ips-on-ubuntu-14-04)
- [Known HAProxy won't start issue](https://discourse.haproxy.org/t/haproxy-wont-start-properly/1394)
- [K8s on Rasbian](https://github.com/teamserverless/k8s-on-raspbian/blob/master/GUIDE.md)
- [Kubernetes High Availability](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/high-availability/)
- [Flannel](https://blog.laputa.io/kubernetes-flannel-networking-6a1cb1f8ec7c)