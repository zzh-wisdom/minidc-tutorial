# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # 1. base image and virtualbox settings
  config.vm.box = "ubuntu/focal64"
  config.vm.box_version = "20200618.0.0"
  config.vm.box_check_update = false
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2
    # patch settings for latest official ubuntu box, reference: https://github.com/hashicorp/vagrant/issues/11777
    vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
    vb.customize ["modifyvm", :id, "--uartmode1", "file", File::NULL]
  end

  # 2. password free ssh
  # 3. hostname and apt source configuration
  # 4. docker and k8s packages
  config.vm.provision "file", source: "conf/", destination: "/home/vagrant/conf"
  config.vm.provision "shell", inline: <<-SHELL
    mv /home/vagrant/conf/config-ssh           /home/vagrant/.ssh/config
    cat /home/vagrant/conf/insecure-key.pub >> /home/vagrant/.ssh/authorized_keys
    mv /home/vagrant/conf/insecure-key         /home/vagrant/.ssh/id_rsa
    chmod 400 /home/vagrant/.ssh/id_rsa

    mv /etc/hosts /etc/hosts.bak
    mv /home/vagrant/conf/hosts /etc/hosts
    mv /etc/apt/sources.list /etc/apt/sources.list.bak
    mv /home/vagrant/conf/sources.list /etc/apt/sources.list

    set -x
    . /vagrant/scripts/k8s-install.sh
    groupadd docker
    usermod -aG docker vagrant
    newgrp docker
  SHELL

  # 5. controller node
  IP0 = 20
  config.vm.define "controller" do |controller|
    controller.vm.network "private_network", ip: "192.168.33.#{IP0}"
    controller.vm.network "forwarded_port", guest: 8001, host: 8001 # access kubernetes dashboard via proxy
    controller.vm.network "forwarded_port", guest: 3000, host: 3000 # access grafana
    controller.vm.network "forwarded_port", guest: 9090, host: 9090 # access prometheus
    controller.vm.hostname = "controller"
  end

  # 6. worker nodes
  (1..2).each do |i|
    config.vm.define "node#{i}" do |node|
      node.vm.network "private_network", ip: "192.168.33.#{i + IP0}"
      node.vm.hostname = "node#{i}"
    end
  end
end
