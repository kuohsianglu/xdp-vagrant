# -*- mode: ruby -*-
# vi: set ft=ruby :

['vagrant-reload'].each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    raise "Vagrant plugin #{plugin} is not installed!"
  end
end

$node_count = 2

libvirt_mgmt_net = "node-mgmt"
libvirt_mgmt_subnet = "10.0.2.0/24"

def nodeIP(id)
    return "192.168.50.#{id+1}"
end

Vagrant.configure('2') do |config|
  config.vm.box = "ubuntu/xenial64" # Ubuntu 16.04

  (1..$node_count).each do |i|
    config.vm.define "node-#{i}" do |node|
      node.vm.hostname = "node-#{i}"
      node_ip = nodeIP(i)
      node.vm.network :private_network, ip: "#{node_ip}"

      node.vm.provider :virtualbox do |vb|
          vb.name = node.vm.hostname
          vb.cpus = "2"
          vb.memory = "2048"
          vb.customize ["modifyvm", :id, "--nicpromisc2", "allow-all"]
          vb.customize ["modifyvm", :id, "--ioapic", "on"]
          vb.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.1", "1"]
          vb.customize ["setextradata", :id, "VBoxInternal/CPUM/SSE4.2", "1"]
          config.vm.synced_folder ".", "/vagrant", type: "rsync"
      end

      node.vm.provider :libvirt do |libvirt|
          libvirt.connect_via_ssh = false
          libvirt.default_prefix = node.vm.hostname
          libvirt.cpus = "2"
          libvirt.memory = "2048"
          libvirt.nic_model_type = "e1000"
          libvirt.management_network_name = "#{libvirt_mgmt_net}"
          libvirt.management_network_address = "#{libvirt_mgmt_subnet}"
      end

      node.vm.provision :shell, :privileged => true, :path => "setup-apt.sh"
      node.vm.provision :shell, :privileged => true, :path => "setup-kernel.sh"
      node.vm.provision :reload
      node.vm.provision :shell, :privileged => true, :path => "setup-bcc.sh"
      node.vm.provision :shell, :privileged => true, :path => "setup-xdp-script.sh"

    end
  end

end

