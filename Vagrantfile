# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  # Ubuntu VM Configuration
  config.vm.define "ubuntu22" do |ubuntu|
    ubuntu.vm.hostname = "ubuntu22"
    ubuntu.vm.box = "gyptazy/ubuntu22.04-arm64"
    ubuntu.vm.provider "vmware_fusion" do |vb|
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  config.vm.define "debian12" do |debian|
    debian.vm.hostname = "debian12"
    debian.vm.box = "gutehall/debian12"
    #config.vm.box_version = "2024.10.07"
    #debian.vm.network "forwarded_port", guest: 80, host: 2201
    debian.vm.provider "vmware_fusion" do |vb|
      vb.memory = 4096
      vb.cpus = 2
    end
  end

  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/playbooks/site.yml"
    ansible.inventory_path = "ansible/inventory.ini"
    #ansible.roles_path = "ansible/roles"
  end

  # Synced folder to share files between host and guest
  config.vm.synced_folder ".", "/vagrant_data"

end
