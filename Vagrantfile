# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = 'centos-65-x64-virtualbox-nocm'
  config.vm.box_url = 'http://puppet-vagrant-boxes.puppetlabs.com/centos-65-x64-virtualbox-nocm.box'

  config.vm.provider 'virtualbox' do |v|
    v.memory = 8192
    v.cpus = 4
  end

  config.vm.network 'forwarded_port', guest: 8088,  host: 8088   # resource manager
  config.vm.network 'forwarded_port', guest: 8042,  host: 8042   # node manager
  config.vm.network 'forwarded_port', guest: 19888, host: 19888  # mapreduce history server
  config.vm.network 'forwarded_port', guest: 50070, host: 50070  # namenode
  config.vm.network 'forwarded_port', guest: 50075, host: 50075  # datanode

#  config.omnibus.chef_version = '12.0.1'
  
end
