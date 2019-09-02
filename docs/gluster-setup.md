# Gluster
[Gluster](https://www.gluster.org/) is a wonderful storage system built to run on commodity hardware. This makes it the perfect solution for a persistent volume file system for kubernetes (you can't get much more "commodity" than a PI). For this example we're just going to use a file system off the root SD card however you may wish to use a USB drive off the USB 3.1 ports in order to save the health of your SD cards and to avoid contention.

NOTE: In the example below we are installing the gluster server on the worker nodes as well. While this allows connectivity to gluster from the kubernetes pods on the worker nodes we really only need the [Gluster Client](https://docs.gluster.org/en/latest/Administrator%20Guide/Setting%20Up%20Clients/) installed. I'll refine the implementation at a later date.

Additional Reference: [Gluster on ARM](https://www.gopeedesignstudio.com/2018/07/13/glusterfs-on-arm/)

## Gluster Setup

Install GlusterFS (every node)
```
> sudo modprobe fuse
> sudo apt-get install -y xfsprogs glusterfs-server
```

Make sure Gluster is started (every node)
```
> sudo systemctl start glusterd
```

Peer all nodes together (run from kuber04m01)
```
> sudo gluster peer probe kuber04m02
> sudo gluster peer probe kuber04m03
> sudo gluster peer probe kuber04w01
> sudo gluster peer probe kuber04w02
> sudo gluster peer probe kuber04w03
> sudo gluster peer probe kuber04w04
```

Validate all peers are connected
```
> sudo gluster peer status

Number of Peers: 6

Hostname: kuber04m03
Uuid: c251f7f7-4682-4ccf-81d1-39bf9961cc12
State: Peer in Cluster (Connected)

Hostname: kuber04w02
Uuid: 27cd9ef1-7218-4d0b-9ad6-46cc187bf026
State: Peer in Cluster (Connected)

Hostname: kuber04m02
Uuid: d70f992b-5295-41b7-9992-38f3be627d98
State: Peer in Cluster (Connected)

Hostname: kuber04w03
Uuid: f5680983-5ad3-4c47-bad7-51bb96b2182e
State: Peer in Cluster (Connected)

Hostname: kuber04w01
Uuid: a71db145-5f43-4412-9da0-8f1de1e6ab1c
State: Peer in Cluster (Connected)

Hostname: kuber04w04
Uuid: 365fc2c9-7b3f-4874-90e5-708387744cc2
State: Peer in Cluster (Connected)

```

## Gluster Volume Setup

Create the first volume directory (all master nodes)
```
> sudo mkdir -p /data/glusterfs/myvol1/brick1/

```

(Optionally format and mount external storage device to above directory here...)

Create the first volume
```
> sudo gluster volume create brick1 kuber04m01:/data/glusterfs/myvol1/brick1/ kuber04m02:/data/glusterfs/myvol1/brick1/ kuber04m03:/data/glusterfs/myvol1/brick1/ 
```

Validate the volume setup

**Make note of the TCP port - you will need it to setup the endpoint in Kubernetes**

```
> sudo gluster volume status

Status of volume: jenkins
Gluster process                             TCP Port  RDMA Port  Online  Pid
------------------------------------------------------------------------------
Brick kuber04m01:/data/glusterfs/myvol1/brick1/    49152     0          Y       15088
Brick kuber04m02:/data/glusterfs/myvol1/brick1/    49152     0          Y       27562
Brick kuber04m03:/data/glusterfs/myvol1/brick1/    49152     0          Y       25493

Task Status of Volume jenkins
------------------------------------------------------------------------------
There are no active volume tasks

```