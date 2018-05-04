hosts = [ { name: 'otustest1', disk1: './server1disk1.vdi', disk2: 'server1disk2.vdi' }]
# { name: 'server2', disk1: './server2disk1.vdi', disk2: 'server2disk2.vdi' },
# { name: 'server3', disk1: './server3disk1.vdi', disk2: 'server3disk2.vdi' }]

Vagrant.configure("2") do |config|

  config.vm.provider :virtualbox do |vb|
   vb.customize ["storagectl", :id, "--add", "sata", "--name", "SATA" , "--portcount", 2, "--hostiocache", "on"]
  end

  hosts.each do |host|

    config.vm.define host[:name] do |node|
      node.vm.hostname = host[:name]
      node.vm.box = "centos/7"
      node.vm.network :public_network
      node.vm.synced_folder "~/vagrant", "/vagrant"
      node.vm.provider :virtualbox do |vb|
        vb.name = host[:name]
        vb.customize ['createhd', '--filename', host[:disk1], '--size', 2 * 1024]
        vb.customize ['createhd', '--filename', host[:disk2], '--size', 2 * 1024]
        vb.customize ['storageattach', :id, '--storagectl', "SATA", '--port', 1, '--device', 0, '--type', 'hdd', '--medium', host[:disk1] ]
        vb.customize ['storageattach', :id, '--storagectl', "SATA", '--port', 2, '--device', 0, '--type', 'hdd', '--medium', host[:disk2] ]
      end
        node.vm.provision "shell", inline: <<-SHELL
            yum install -y vim mdadm wget gzip uzip
            
            
            
         SHELL
    end

  end

end
