# -*- mode: ruby -*-
# vi: set ft=ruby :

## if you change these, change vagrant_hosts too
#
# would _dearly_ love to make this less clunkable,
# and just read vagrant_hosts directly.

hosts = [
  {:name => "mesos-master1", :ip => "10.0.0.103",   :ram => 768},
  {:name => "mesos-slave1",  :ip => "10.0.0.104",   :ram => 1250},
  {:name => "mesos-slave2",  :ip => "10.0.0.105",   :ram => 1250},
]

Vagrant.configure("2") do |config|

  # setup /etc/hosts on each vm
  # - requires the vagrant-hostmanager plugin
  config.hostmanager.enabled = true
  # set to 'false' if you don't wan't to manage _your_ /etc/hosts
  config.hostmanager.manage_host = true
  config.hostmanager.include_offline = true

  hosts.each do |host|
    config.vm.define host[:name] do |c|

      c.vm.box = "centos7-core"
      c.vm.box_url =
        "https://github.com/rasputnik/centos7-packer/releases/download/v0.1/CentOS-7.0-1406-x86_64-v20140721-virtualbox.box"

      c.vm.hostname = host[:name]

      c.vm.network :private_network, ip: host[:ip], netmask: "255.255.255.0"

      c.vm.provider("virtualbox") do |vb|
        vb.memory = host[:ram]
      end

      # turn off shared folder
      c.vm.synced_folder ".", "/vagrant", :disabled => true
    end
  end
end
