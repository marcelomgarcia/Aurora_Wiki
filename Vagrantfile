# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  config.vm.box = "centos/7"
  config.vm.network :forwarded_port, guest: 80, host: 4080
  config.vm.network :forwarded_port, guest: 81, host: 4081

  config.vm.provider "virtualbox" do |vv|
    vv.memory = 2048
    vv.cpus = 2
  end
  #config.vm.provision :shell, path: "bootstrap.sh"
  config.vm.hostname = "cancer"
  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "playbook.yaml"
  end
  
  #
  # The End
  # 
end
