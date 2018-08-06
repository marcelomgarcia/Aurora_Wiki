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

I wanted the Wiki to be in a directory independent of the version of Dokuwiki, and to do this, I created a symbolic link from the directory created by the extraction of the file named `wiki`

    [vagrant@cancer ~]$ ls -l /var/www/html/wiki
    lrwxrwxrwx. 1 apache apache 35 Aug  3 13:07 /var/www/html/wiki -> /var/www/html/dokuwiki-2018-04-22a
    [vagrant@cancer ~]$

In the Ansible this was done registering the extraction of the file, and then using when the symbolic link was created

    - name: Extract Dokuwiki
      unarchive:
    (...)
        list_files: yes
      register: archive_contents
    (...)
    - name: Symbolic link from Dokuwiki to wiki
      file:
        src: "/var/www/html/{{ archive_contents.files[0] }}"
    (...)

The Apache configuration was done by creating a directory in `/etc/httpd/`, and copying the configuration file

    - name: Wiki configuration directory
      file:
        path: /etc/httpd/vhosts.d/
    (...)
        state: directory

    - name: Copy wiki configuration file.
      copy:
        src: files/wiki.conf
        dest: /etc/httpd/vhosts.d/
    (...)

Finally we reboot the server

    - name: Reboot the server
        command: /sbin/shutdown -r +1
        async: 0
        poll: 0
        ignore_errors: true

To access the configuration page of the new wiki, go to the URL

    http://localhost:<port>/install.php

The *port* was defined in the `Vagrantfile`. Here we use the value 4080.

## How to use

These are the basic Vagrant commands:

* To start (or install if necessary) the virtual machine: `vagrant up`

* To gracefully shutdown the machine: `vagrant halt`

* Force the provisioning: `vagrant provision`

* To access the virtual machine: `vagrant ssh`

* To erase the virtual machine: `vagrant destroy`.

**Note**, to use vagrant on Windows 7, I had to upgrade the Powershell, from version "2" to version "5".