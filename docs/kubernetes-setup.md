# Kubernetes Setup

## First Master Node (kuber04m01)

Perform the initial Kubernetes setup. As before, change the controlPlaneEndpoint to match your setup.

```
> sudo nano kubeadm-config.yaml
# paste the following

apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "192.168.200.249:443"
networking:
  podSubnet: 10.244.0.0/16
apiServer:
  certSANs:
  - kubernetes
```

Initialize the Kubernetes cluster
```
> sudo kubeadm init --config=kubeadm-config.yaml --upload-certs -v 4
```

Configure your local kubectl. If for some reason you have to start over and issue a **kubeadm reset** you will need to do this again. **Make note of the join commands** that are presented once this command completes.

```
# from ~

> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Install [Flannel](https://coreos.com/flannel/docs/latest/) as the networking component of the cluster
```
> kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

```

Copy the cluster certs to the other master nodes
```
> sudo apt install sshpass

> sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/ca.crt kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/ca.key kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/sa.key kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/sa.pub kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/front-proxy-ca.crt kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/front-proxy-ca.key kadmin@kuber04m02: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/etcd/ca.crt kadmin@kuber04m02:etcd-ca.crt && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/etcd/ca.key kadmin@kuber04m02:etcd-ca.key

> sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/ca.crt kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/ca.key kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/sa.key kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/sa.pub kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/front-proxy-ca.crt kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/front-proxy-ca.key kadmin@kuber04m03: && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/etcd/ca.crt kadmin@kuber04m03:etcd-ca.crt && sudo sshpass -p "<your password>" scp /etc/kubernetes/pki/etcd/ca.key kadmin@kuber04m03:etcd-ca.key
```

## Master Nodes 2 and 3 (kuber04m02 & kuber04m03)

Move the certs to the /etc/kubernets/pki path
```
sudo mv ca.crt /etc/kubernetes/pki && sudo mv ca.key /etc/kubernetes/pki && sudo mv sa.key /etc/kubernetes/pki && sudo mv sa.pub /etc/kubernetes/pki && sudo mv front-proxy-ca.crt /etc/kubernetes/pki && sudo mv front-proxy-ca.key /etc/kubernetes/pki && sudo mkdir /etc/kubernetes/pki/etcd && sudo cp etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt && sudo cp etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
```

Join the master nodes to 01 using the join command from when you ran **kubeadm init**. Make sure it's the command with **--controlplane** in it.

To regenerate the join commands use:
```
> sudo kubeadm token create --print-join-command
```

Be sure to do the following after the join succeeds
```
> mkdir -p $HOME/.kube
> sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
> sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## All Worker Nodes

Join the worker nodes to 01 using the join command from when you ran **kubeadm init**. Make sure the command does **NOT** have **--controlplane** in it.

To regenerate the join commands use:
```
> sudo kubeadm token create --print-join-command
```

## Confirmation

You should be able to see the following response from kuber04m01
```
> kubectl get nodes

NAME         STATUS   ROLES    AGE   VERSION
kuber04m01   Ready    master   24h   v1.15.1
kuber04m02   Ready    master   24h   v1.15.1
kuber04m03   Ready    master   24h   v1.15.1
kuber04w01   Ready    <none>   23h   v1.15.2
kuber04w02   Ready    <none>   23h   v1.15.2
kuber04w03   Ready    <none>   23h   v1.15.2
kuber04w04   Ready    <none>   23h   v1.15.3
```
