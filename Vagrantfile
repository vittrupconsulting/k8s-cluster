Vagrant.configure("2") do |config|

  config.vm.box = "ubuntu/jammy64"
  config.vm.provision "shell", inline: <<-SHELL
	sudo sed -e "s/^.*${HOSTNAME}.*/127.0.0.1 ${HOSTNAME} ${HOSTNAME}.local/" -i /etc/hosts
	sudo sed -e '/^.*ubuntu-bionic.*/d' -i /etc/hosts
	sudo sed -i -e 's/#DNS=/DNS=8.8.8.8/' /etc/systemd/resolved.conf
	echo "192.168.33.13 master" | sudo tee --append /etc/hosts
    echo "192.168.33.14 worker-1" | sudo tee --append /etc/hosts
    echo "192.168.33.15 worker-2" | sudo tee --append /etc/hosts
	sudo apt-get update
    sudo apt-get install containerd -y
	sudo mkdir -p /etc/containerd
	sudo containerd config default /etc/containerd/config.toml
	sudo apt-get update
	sudo apt-get install -y apt-transport-https gnupg2
	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
	echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee --append /etc/apt/sources.list.d/kubernetes.list
	sudo apt-get update
	sudo apt-get install -y kubelet kubeadm kubectl
	sudo apt-get install bash-completion
	kubectl completion bash | sudo tee --append /etc/bash_completion.d/kubectl
	echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee --append /etc/sysctl.d/k8s.conf
	echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee --append /etc/sysctl.d/k8s.conf
	echo '1' | sudo tee /proc/sys/net/ipv4/ip_forward
	sudo sysctl --system
	sudo modprobe overlay
	sudo modprobe br_netfilter
	sudo service systemd-resolved restart
   SHELL

  config.vm.define "master" do | w |
  w.vm.hostname = "master"
  w.vm.network "private_network", ip: "192.168.33.13"
  w.vm.provider "virtualbox" do |vb|
    vb.memory = "4096"
    vb.cpus = 2
    vb.name = "master"
  end
  #mkdir -p /home/vagrant/.kube
  #sudo cp /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  #sudo chown vagrant:vagrant /home/vagrant/.kube/config
  #kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  end

  config.vm.box = "ubuntu/jammy64"
  config.vm.define "worker-1" do | w |
  w.vm.hostname = "worker-1"
  w.vm.network "private_network", ip: "192.168.33.14"
  w.vm.provider "virtualbox" do |vb|
	vb.memory = "2048"
	vb.cpus = 2
	vb.name = "worker-1"
  end
  end

  config.vm.box = "ubuntu/jammy64"
  config.vm.define "worker-2" do | w |
  w.vm.hostname = "worker-2"
  w.vm.network "private_network", ip: "192.168.33.15"
  w.vm.provider "virtualbox" do |vb|
	vb.memory = "2048"
	vb.cpus = 2
	vb.name = "worker-2"
  end
  end
end
