# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  (1..2).each do |i|
    config.vm.define "worker-#{i}" do |node|
      node.vm.box = "ubuntu/focal64"
      node.vm.hostname = "worker-#{i}"
      node.vm.network "public_network"
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "4096"
      end
      node.vm.provision "shell", inline: <<-SHELL
        apt-get update
        apt-get upgrade -y
        echo 'root:*****' | chpasswd
        sed -i 's/.*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
        systemctl restart sshd
        ufw disable
      SHELL
    end
  end

  config.vm.define "master" do |node|
    node.vm.box = "ubuntu/focal64"
    node.vm.hostname = "master"
    node.vm.network "public_network"
    node.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
    end
    node.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get upgrade -y
      echo 'root:*****' | chpasswd
      sed -i 's/.*PermitRootLogin.*/PermitRootLogin yes/' /etc/ssh/sshd_config
      systemctl restart sshd
      ufw disable
    SHELL
  end
end

