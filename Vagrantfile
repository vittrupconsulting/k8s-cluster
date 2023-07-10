Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"

  config.vm.define "common", autostart: false do |common|
    common.vm.provision "shell", inline: <<-SHELL
      rm --force /vagrant/id_rsa
      rm --force /vagrant/id_rsa.pub
      ssh-keygen -t rsa -q -f /vagrant/id_rsa -N "" -C "ansible"
    SHELL
  end

#  config.vm.provision "shell", inline: <<-SHELL
#	sudo sed -e "s/^.*${HOSTNAME}.*/127.0.0.1 ${HOSTNAME} ${HOSTNAME}.local/" -i /etc/hosts
#	sudo sed -e '/^.*ubuntu-jammy.*/d' -i /etc/hosts
##	  sudo sed -i -e 's/#DNS=/DNS=8.8.8.8/' /etc/systemd/resolved.conf
##    echo "192.168.33.13 master" | sudo tee --append /etc/hosts
##    echo "192.168.33.14 worker-14" | sudo tee --append /etc/hosts
##    echo "192.168.33.15 worker-15" | sudo tee --append /etc/hosts
#	sudo apt-get update
#    sudo apt-get install containerd -y
#	sudo mkdir -p /etc/containerd
#	sudo containerd config default /etc/containerd/config.toml
#	sudo apt-get update
#	sudo apt-get install -y apt-transport-https gnupg2
#	curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
#	echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee --append /etc/apt/sources.list.d/kubernetes.list
#	sudo apt-get update
#	sudo apt-get install -y kubelet kubeadm kubectl
#	sudo apt-get install bash-completion
#	kubectl completion bash | sudo tee --append /etc/bash_completion.d/kubectl
#	echo "net.bridge.bridge-nf-call-ip6tables = 1" | sudo tee --append /etc/sysctl.d/k8s.conf
#	echo "net.bridge.bridge-nf-call-iptables = 1" | sudo tee --append /etc/sysctl.d/k8s.conf
#	echo '1' | sudo tee /proc/sys/net/ipv4/ip_forward
#	sudo sysctl --system
#	sudo modprobe overlay
#	sudo modprobe br_netfilter
#	sudo service systemd-resolved restart
#   SHELL

  config.vm.define "master" do | w |
  w.vm.hostname = "master"
  w.vm.network "private_network", ip: "192.168.33.13"
  w.vm.provider "virtualbox" do |vb|
    vb.memory = "6144"
    vb.cpus = 2
    vb.name = "master"
  end
  w.vm.provision "shell", inline: <<-SHELL
    cat /vagrant/id_rsa.pub | tee --append /home/vagrant/.ssh/authorized_keys
  SHELL

  #sudo kubeadm init --apiserver-advertise-address 192.168.33.13 --pod-network-cidr=10.244.0.0/16
  #mkdir -p /home/vagrant/.kube
  #sudo cp /etc/kubernetes/admin.conf /home/vagrant/.kube/config
  #sudo chown vagrant:vagrant /home/vagrant/.kube/config
  #kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
  end

(14..15).each do |i|
  config.vm.define "worker-#{i}" do |node|
  node.vm.hostname = "worker-#{i}"
  node.vm.network "private_network", ip: "192.168.33.#{i}"
  node.vm.provider "virtualbox" do |vb|
	vb.memory = "2048"
	vb.cpus = 2
	vb.name = "worker-#{i}"
  end
  end
  #kubeadm token create --print-join-command
  end

  config.vm.define "ansible", autostart: true do |ansible|
    ansible.vm.hostname = "ansible"
    ansible.vm.network "private_network", ip: "192.168.33.12"
    ansible.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
      vb.cpus = 2
    end
    ansible.vm.provision "shell", inline: <<-SHELL
      cp /vagrant/id_rsa /home/vagrant/.ssh/id_rsa
      chmod 600 /home/vagrant/.ssh/id_rsa
      sudo apt-get update
      sudo apt-get install --yes python3-pip
      pip install ansible
    SHELL
    ansible.vm.provision "shell", inline: <<-SHELL
      export ANSIBLE_HOST_KEY_CHECKING=False
      ansible --become --inventory "192.168.33.13,192.168.33.14,192.168.33.15," --module-name "ansible.builtin.lineinfile" --args "path='/etc/systemd/resolved.conf' regexp='^#DNS=' line='DNS=8.8.8.8'" all
      ansible --become --inventory "192.168.33.13,192.168.33.14,192.168.33.15," --module-name "ansible.builtin.lineinfile" --args "path='/etc/hosts' regexp='^192.168.33.13' line='192.168.33.13 master'" all
      ansible --become --inventory "192.168.33.13,192.168.33.14,192.168.33.15," --module-name "ansible.builtin.lineinfile" --args "path='/etc/hosts' regexp='^192.168.33.14' line='192.168.33.14 worker-14'" all
      ansible --become --inventory "192.168.33.13,192.168.33.14,192.168.33.15," --module-name "ansible.builtin.lineinfile" --args "path='/etc/hosts' regexp='^192.168.33.15' line='192.168.33.15 worker-15'" all
    SHELL
  end

end
