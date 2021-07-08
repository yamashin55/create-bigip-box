# create-bigip-box

1. download ova file from download.f5.com
2. create box
  ```
  OVA_FILE=BIGIP-16.1.0-0.0.19.ALL_1SLOT-vmware.ova
  BOX_FILE=BIGIP-16.1.0.box
  VBoxManage import ${OVA_FILE} --vsys 0 --memory 2048 --cpus 2 --eula accept
  VBoxManage list vms
  VM_NAME=vm
  vagrant package --base ${VM_NAME} --output ${BOX_FILE}
  VBoxManage unregistervm  vm --delete
  VBoxManage list vms
  vagrant box add BIGIP-16.1.0.box --name BIGIP-16.1.0
  ```
3. create Vagrantfile
 ```
 vagrant init -m bigip
 vi Vagrantfile
 
# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = 2
ENV['BIGIP_LICENSE'] ||= ''
ENV['BIGIP_PORT_443'] ||= '11443'
ENV['BIGIP_NAME'] ||= 'bigip'
ENV['BOX_URL'] ||=  'bigip'
#ENV['BOX_URL'] ||=  'BIGIP-16.1.0'

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.boot_timeout = 10
  config.ssh.keys_only = false
  config.ssh.password = "default"
  config.ssh.username = "root"
  config.ssh.connect_timeout = 2

  config.vm.define ENV['BIGIP_NAME'] do |v|
    v.vm.box = ENV['BOX_URL']
    # BIG-IP cannot mount shares in Virtualbox because Guest-Additions
    # cannot be installed on it.
    v.vm.synced_folder ".", "/vagrant", disabled: true
    v.vm.network :forwarded_port, guest: 443, host: ENV['BIGIP_PORT_443']
    v.vm.network :private_network, ip: "192.168.50.20", auto_config: false
    v.vm.network :private_network, ip: "10.2.3.2", auto_config: false

    v.vm.provider :virtualbox do |p|
      # Required for >8 network interfaces
      p.customize ["modifyvm", :id, "--ioapic", "off"]
      p.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
      p.customize ["modifyvm", :id, "--memory", 2048]
      p.customize ["modifyvm", :id, "--cpus", 2]
      p.customize ["modifyvm", :id, "--name", ENV['BIGIP_NAME']]

      # NICs need to be virtio because BIG-IP doesn't have drivers for
      # the others
      p.customize ["modifyvm", :id, "--nic1", "nat"]
      p.customize ["modifyvm", :id, "--nictype1", "virtio"]
      p.customize ["modifyvm", :id, "--nic2", "hostonly"]
      p.customize ["modifyvm", :id, "--nictype2", "virtio"]
      p.customize ["modifyvm", :id, "--nic3", "hostonly"]
      p.customize ["modifyvm", :id, "--nictype3", "virtio"]
    end
  end
end
 ```
5. vagrant up
   ```
   vagrant up
   ```
