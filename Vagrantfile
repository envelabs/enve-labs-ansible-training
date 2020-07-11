# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  config.vm.define "master" do |subconfig|
    subconfig.vm.box = "hashicorp/bionic64"
    subconfig.vm.hostname = "master"

  	subconfig.vm.provider :virtualbox do |vb|
    	vb.name = "master"
  	end

  	subconfig.vm.network :public_network, bridge: 'en0: Wi-Fi (AirPort)'
    subconfig.vm.provision "shell",
      inline: "echo Hello, from master"

		config.vm.provision "shell", inline: <<-SHELL
		sudo apt -y update
		sudo apt install -y software-properties-common
		sudo apt-add-repository -y ppa:ansible/ansible
		sudo apt -y update
		sudo apt install -y ansible
		SHELL
  end

  config.vm.define "node1" do |subconfig|
    subconfig.vm.box = "hashicorp/bionic64"
    subconfig.vm.hostname = "node1"

  	subconfig.vm.provider :virtualbox do |vb|
    	vb.name = "node1"
  	end

  	subconfig.vm.network :public_network, bridge: 'en0: Wi-Fi (AirPort)'
  	subconfig.vm.network "forwarded_port", guest: 80, host: 80
    subconfig.vm.provision "shell",
      inline: "echo Hello, from node1"

    subconfig.vm.provision "ansible_local" do |ansible|
      ansible.config_file = 'ansible.cfg'
      ansible.inventory_path = "inventory"
      ansible.playbook = "play.webserver.yml"
    end

  end

  config.vm.define "node2" do |subconfig|
    subconfig.vm.box = "hashicorp/bionic64"
    subconfig.vm.hostname = "node2"

  	subconfig.vm.provider :virtualbox do |vb|
    	vb.name = "node2"
  	end

  	subconfig.vm.network :public_network, bridge: 'en0: Wi-Fi (AirPort)'
  	subconfig.vm.network "forwarded_port", guest: 27017, host: 27017
    subconfig.vm.provision "shell",
      inline: "echo Hello, from node2"
    subconfig.vm.provision "ansible_local" do |ansible|
      ansible.config_file = 'ansible.cfg'
      ansible.inventory_path = "inventory"
      ansible.playbook = "play.dbserver.yml"
    end
  end

end
