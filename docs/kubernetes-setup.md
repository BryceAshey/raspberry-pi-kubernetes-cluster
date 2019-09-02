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

## Master Nodes 2 and 3 (kuber04m02 & kuber04m03)

## All Worker Nodes