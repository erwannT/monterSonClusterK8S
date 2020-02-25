# Init

## Provisionner les VMs

```
vagrant up

# connexion a unes des instances
vagrant ssh k8s1
vagrant ssh k8s2
vagrant ssh k8s3

```

Vagrant va provisionner 3 noeuds.

Pour la suite, le premier sera utilisé en tant que master, les autres en tant que workers.

## Precharger les images

```

sudo kubeadm config images pull
```

# Init du master avec flannel

```
sudo kubeadm init --apiserver-advertise-address=172.18.18.2 --pod-network-cidr=10.244.0.0/16

kubect  apply -f kube-flannel.yaml

# suivre l'installation de flannel
watch kubectl get pods -n kube-system
```

Manifeste original de Flannel : [Flannel](https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml)

L'option --iface=enp0s8 a été ajouté a chaque noeud afin d'utiliser l'interface reseau enp0s8

# Init du master avec calico

https://docs.projectcalico.org/v3.9/reference/node/configuration

```
sudo kubeadm init --apiserver-advertise-address=172.18.18.2 --pod-network-cidr=192.168.0.0/16

kubect  apply -f calico.yaml

# suivre l'installation de calico
watch kubectl get pods -n kube-system
```

Les variables d'environnements suivantes ont été ajoutés afin d'utiliser l'interface reseau enp0s8

- name: IP_AUTODETECTION_METHOD
  value: interface=enp0s8
- name: IP6_AUTODETECTION_METHOD
  value: interface=enp0s8

Manifeste original de Calico : [Calico](https://docs.projectcalico.org/v3.11/manifests/calico.yaml)

# Preparation de kubectl sur le master

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

```

# Association des workers

[Plus d'info](https://kubernetes.io/fr/docs/setup/independent/create-cluster-kubeadm/#join-nodes)

La commande ci-dessous est fourni dans les logs de l'initialisation du master.

```
sudo kubeadm join 172.18.18.2:6443 --token <generated token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

# Nettoyage

```

vagrant destroy

```

# Astuces

```
vagrant snapshot save <name>
vagrant snapshot restore <name>
```

# Sources

[Solution avec flannel](https://medium.com/@anilkreddyr/kubernetes-with-flannel-understanding-the-networking-part-1-7e1fe51820e4) : article medium montrant le problème des deux interfaces avec virtualbox

[Les different plugin CNI](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)
