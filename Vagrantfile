Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
  end

  $script = <<-SCRIPT
  echo "172.18.18.2 k8s1" >> /etc/hosts
  echo "172.18.18.3 k8s2" >> /etc/hosts
  echo "172.18.18.4 k8s3" >> /etc/hosts


  echo 'libssl1.1 libraries/restart-without-asking boolean true' | sudo debconf-set-selections

  sudo apt-get update && sudo apt-get  -y install apt-transport-https ca-certificates curl software-properties-common 
  curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
  add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs) stable"
  apt-get update && apt-get install docker-ce=18.06.2~ce~3-0~ubuntu -y

  cat > /etc/docker/daemon.json <<EOF
  {
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
      "max-size": "100m"
    },
    "storage-driver": "overlay2"
  }
EOF

  mkdir -p /etc/systemd/system/docker.service.d

  sudo systemctl daemon-reload
  sudo systemctl restart docker

  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

  cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
  deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF

  sudo apt-get update && sudo apt-get install kubelet=1.16.3-00 kubeadm=1.16.3-00 kubectl=1.16.3-00 -y

  apt-mark hold docker-ce kubelet kubeadm kubectl

  cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

  sudo sysctl -p

  SCRIPT



  config.vm.provision "shell", inline: $script

  config.vm.define "k8s1" do |k8s1|
    k8s1.vm.hostname = "k8s1"
    k8s1.vm.network "private_network",  ip: "172.18.18.2"
    k8s1.vm.provision "file", source: "calico.yaml", destination: "calico.yaml"
    k8s1.vm.provision "file", source: "kube-flannel.yaml", destination: "kube-flannel.yaml"
  end

  config.vm.define "k8s2" do |k8s2|
    k8s2.vm.hostname = "k8s2"
    k8s2.vm.network "private_network",  ip: "172.18.18.3"
  end

  config.vm.define "k8s3" do |k8s3|
    k8s3.vm.hostname = "k8s3"
    k8s3.vm.network "private_network",  ip: "172.18.18.4"
  end
end