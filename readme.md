source : 

Solution avec flannel https://medium.com/@anilkreddyr/kubernetes-with-flannel-understanding-the-networking-part-1-7e1fe51820e4

Les different plugin CNI : https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/


# init

Precharger les images pour gagner du temps
```
kubeadm config images pull
```


# Preparation pour flannel

```
sudo kubeadm init --apiserver-advertise-address=172.18.18.2 --pod-network-cidr=10.244.0.0/16

wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml

```


Ajouter a la commande du container de kube-flannel : 

\- --iface=enp0s8





# Preparation pour calico

https://docs.projectcalico.org/v3.9/reference/node/configuration

```
sudo kubeadm init --apiserver-advertise-address=172.18.18.2 --pod-network-cidr=192.168.0.0/16

wget https://docs.projectcalico.org/v3.11/manifests/calico.yaml

Env specifique

- name:  IP_AUTODETECTION_METHOD
  value: interface=enp0s8
- name:  IP6_AUTODETECTION_METHOD
  value: interface=enp0s8


```


# attention

la suppression du cluster k8s ne restore pas les parametres reseau