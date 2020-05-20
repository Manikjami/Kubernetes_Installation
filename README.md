# Kubernetes Installation on Ubuntu 20.04

[Docker](https://docs.docker.com/)

[Kubernetes](https://kubernetes.io/)

### Set up the Docker and Kubernetes repositories:

> Download the GPG key for docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

> Add the docker repository
```
# Docker still does not have the stable release for Ubuntu 20.04, so we will be using the bionic repository.
# we can get the latest release versions from https://docs.docker.com

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

```

> Add the GPG key for kubernetes
```
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
```

> Add the kubernetes repository

__Check for the latest release in https://packages.cloud.google.com/apt/dists__
```
cat << EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

> Update the repository
```
sudo apt-get update
```


> Install Docker and Kubernetes packages.

__Note that if you want to use a newer version of Kubernetes, change the version installed for kubelet, kubeadm, and kubectl and be sure that all three use the same version.
These version should support the Docker CE version.__
```

# Use the same versions to avoid issues with the installation.
sudo apt-get install -y docker-ce=5:19.03.8~3-0~ubuntu-bionic kubelet=1.18.2-00 kubeadm=1.18.2-00 kubectl=1.18.2-00


```

> To hold the versions so that the versions will not be upgraded.
```
sudo apt-mark hold docker-ce kubelet kubeadm kubectl
```


> Enable the iptables bridge
Set a a value in the sysctl file , to allow proper network settings for Kubernetes on the servers.
```
echo "net.bridge.bridge-nf-call-iptables=1" | sudo tee -a /etc/sysctl.conf
```

> To make the changes to take immediate effect for the iptables
```
sudo sysctl -p
```

### On the Kube master server

> Initialize the cluster by passing the cidr value and the value will depend on the type of network CLI you choose.

__Use either Flannel or Calico__
```
# For flannel network
# Copy your join command and keep it safe.
sudo kubeadm init --pod-network-cidr=10.244.0.0/16

# Copy your join command and keep it safe.
# Below is a sample
kubeadm join 10.128.0.2:6443 --token swi0ci.jq9l75eg8lvpxz6g --discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b4d1
```

```
# For Calico networl
# Make sure to copy the join command
sudo kubeadm init --pod-network-cidr=192.168.0.0/16

# Copy your join command and keep it safe.
# Below is a sample
kubeadm join 10.128.0.2:6443 --token swi0ci.jq9l75eg8lvpxz6g --discovery-token-ca-cert-hash sha256:2c3cdfa898334b0dfc0f73bbccb998d03f61252ee50f0405c85ba735ff90b4d1
```

> To start using the cluster with current user.
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


> Set the flannel networking
```
# Use this if you have initialised the cluster with Flannel network add on.
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

> To set up the Calico network
```
# Use this if you have initialised the cluster with Calico network add on.
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

> Check the nodes
```
kubectl get nodes
```

### On each of Kube node server

> Joining the node to the cluster:
```
sudo kubeadm join $controller_private_ip:6443 --token $token --discovery-token-ca-cert-hash $hash
```

**TIP**

> If the code is lost, we can retrieve using below command
```
kubeadm token create --print-join-command
```

