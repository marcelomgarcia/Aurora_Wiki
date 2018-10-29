# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.network :forwarded_port, guest: 80, host: 4080

  config.vm.provider "virtualbox" do |vv|
    vv.memory = 2048
    vv.cpus = 4
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
