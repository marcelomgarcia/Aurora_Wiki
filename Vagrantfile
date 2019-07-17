# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://172.29.208.130:8080"
    config.proxy.https    = "http://172.29.208.130:8080"
    config.proxy.no_proxy = "localhost, 127.0.0.1, .dwd.de"
  end 
  config.vm.box = "centos/7"
  config.vm.network :forwarded_port, guest: 80, host: 4080

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
