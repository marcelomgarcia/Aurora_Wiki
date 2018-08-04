# Creating a Wiki

Installation and configuration of a Wiki. The project starts with a Vagrant file that will create a virtual machine running CentOS 7. On this machine we will install Apache and extract the stable version of Dokuwiki.

## Vagrant

The project uses Vagrant to create and management of the virtual machine. The machine has 4GB and 4 processors

    config.vm.provider "virtualbox" do |vv|
      vv.memory = 4096
      vv.cpus = 4
    end

The virtual machine will forward the ports "80" as port "4080" on the host

    config.vm.network :forwarded_port, guest: 80, host: 4080

So we can access the Wiki on the host.

Finally we configure Vagrant to run an Ansible play called `playbook.yaml`

    config.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "playbook.yaml"
    end
This play will configure the new machine by installing extra packages, and extracting the tar file with the latest stable version of Dokuwiki.

## Ansible

The Vagrant file does the basic set up, to install and configure all other componentes, I use Ansible. The playbook install extra packages like: development tools, kernel headers (not actually used since I'm not building the guest part), LSB, etc. The next step is to install and enable Apache. Then installing the EPEL repository for convinience.

The Dokuwiki require a newer verion of PHP than provided by CentOS 7. So we need another source. I use the "remi" repository. I choose the latest stable version available, which is 7.2 at this moment.

The Dokuwiki package is downloaded directly from the web site as a compressed file. The file is downloaded to the `/tmp` directory. Once downloaded, it is extracted in the final destination and with permissions to the *apache* user and group. During the installation of the Dokuwiki, there was a message of Ansible(?) not being able to extract the tar ball file:

    fatal: [default]: FAILED! => {"changed": false, "msg": "Failed to find handler for \"files/dokuwiki-stable.tgz\". Make sure the required command to extract the file is installed. Command \"/bin/gtar\" could not handle archive. Command \"/bin/unzip\" could not handle archive."}
        to retry, use: --limit @/vagrant/playbook.retry

Following the entry for bug [#35645](https://github.com/ansible/ansible/issues/35645), someone mentioned that downloading to the "/tmp" and then extracting the file solved the problem. And that is how I also solved the problem.
