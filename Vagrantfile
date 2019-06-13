# Defines our Vagrant environment
#
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  ssh_pub_key = File.readlines("../ssh/id_rsa.pub").first.strip

  $disable_ipv6 = <<-SCRIPT
	echo "net.ipv6.conf.all.disable_ipv6 = 1
		  net.ipv6.conf.default.disable_ipv6 = 1
		  net.ipv6.conf.lo.disable_ipv6 = 1
		  net.ipv6.conf.eth0.disable_ipv6 = 1" >> /etc/sysctl.conf
	sysctl -p
  SCRIPT

  config.vm.box_check_update = true

  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = true
  end

  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = false
  end

  if Vagrant.has_plugin?("vagrant-hosts")
    config.vm.provision :hosts do |provisioner|
      provisioner.autoconfigure = true
      provisioner.sync_hosts = true
      provisioner.add_localhost_hostnames = false

      provisioner.imports = ['virtualbox']
      provisioner.exports = {
        'virtualbox' => [
          ['@vagrant_private_networks', ['@vagrant_hostnames']],
        ],
      }
    end
  end

  config.vm.provision :shell, :inline => "grep -q '#{ssh_pub_key}' ~/.ssh/authorized_keys || echo '#{ssh_pub_key}' >> ~/.ssh/authorized_keys", privileged: false
 
  # create management node
  config.vm.define "master-elk.local" do |config|
    config.vm.box = "centos/7"

    config.vm.hostname = "master-elk.local"
    config.vm.define "master-elk.local"
    config.vm.network :private_network, ip: "192.168.3.10", hostsupdater: "skip"
    config.vm.network :forwarded_port, guest: 5601, host: 5601, hostsupdater: "skip"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "4096"
      vb.name = "master-elk.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
	
	config.vm.provision :shell, :inline => "$disable_ipv6"
  end

# create postgresql node
  config.vm.define "master-postgresql.local" do |config|
    config.vm.box = "centos/7"

    config.vm.hostname = "master-postgresql.local"
    config.vm.define "master-postgresql.local"
    config.vm.network :private_network, ip: "192.168.3.11", hostsupdater: "skip"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "master-postgresql.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
	
	config.vm.provision :shell, :inline => "$disable_ipv6"
  end
  
  # create docker node
  config.vm.define "master-docker.local" do |config|
    config.vm.box = "centos/7"

    config.vm.hostname = "master-docker.local"
    config.vm.define "master-docker.local"
    config.vm.network :private_network, ip: "192.168.3.12", hostsupdater: "skip"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "master-docker.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
	
	config.vm.provision :shell, :inline => "$disable_ipv6"
  end
  
  # create docker node
  config.vm.define "master-nginx.local" do |config|
    config.vm.box = "centos/7"

    config.vm.hostname = "master-nginx.local"
    config.vm.define "master-nginx.local"
    config.vm.network :private_network, ip: "192.168.3.13", hostsupdater: "skip"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "master-nginx.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end
	
	config.vm.provision :shell, :inline => "$disable_ipv6"
  end
  
  # create ansible node
  config.vm.define "ansible-elk.local" do |config|
    config.vm.box = "centos/7"

    config.vm.hostname = "ansible-elk.local"
    config.vm.define "ansible-elk.local"
    config.vm.network :private_network, ip: "192.168.3.2", hostsupdater: "skip"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "1024"
      vb.name = "ansible-elk.local"

      vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
    end

    config.vm.synced_folder ".", "/vagrant", type:"virtualbox", disabled: false

    config.vm.provision :shell, :inline  => 'rm -f ~/.ssh/id_rsa ~/.ssh/id_rsa.pub >/dev/null 2>&1', privileged: false
    config.vm.provision "file", source: "../ssh/id_rsa", destination: "~/.ssh/id_rsa"
    config.vm.provision "file", source: "../ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
    config.vm.provision :shell, :inline  => 'chmod 600 ~/.ssh/id_rsa ~/.ssh/id_rsa.pub', privileged: false
    config.vm.provision :shell, :inline  => "$disable_ipv6"

    config.vm.provision "ansible_local" do |ansible|
      ansible.install_mode = "pip_args_only"
      ansible.pip_args = "ansible==2.4.3"
      ansible.version = "2.4.3.0"
      ansible.limit = ""
      ansible.verbose = false
      ansible.playbook = "provision.yml"
      ansible.host_vars = {
        "master-elk.local" => {
          "ansible_user" => "vagrant",
          "ansible_host" => "master-elk.local",
          "ansible_ssh_private_key_file" => "~/.ssh/id_rsa"
        },
        "master-postgresql.local" => {
          "ansible_user" => "vagrant",
          "ansible_host" => "master-postgresql.local",
          "ansible_ssh_private_key_file" => "~/.ssh/id_rsa"
        },
        "master-docker.local" => {
          "ansible_user" => "vagrant",
          "ansible_host" => "master-docker.local",
          "ansible_ssh_private_key_file" => "~/.ssh/id_rsa"
        },			
        "master-nginx.local" => {
          "ansible_user" => "vagrant",
          "ansible_host" => "master-nginx.local",
          "ansible_ssh_private_key_file" => "~/.ssh/id_rsa"
        },
        "ansible-elk.local" => {
          "ansible_user" => "vagrant",
          "ansible_host" => "ansible-elk.local",
          "ansible_ssh_private_key_file" => "~/.ssh/id_rsa"
        }
      }
      ansible.groups = {
        "postgresql" => ["master-postgresql.local"],
		"docker" => ["master-docker.local"],
		"nginx" => ["master-nginx.local"],
		"elk" => ["master-elk.local"],
        "ansible" => ["ansible-elk.local"]
      }
    end
  end
end
