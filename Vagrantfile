# -*- mode: ruby -*-
# vi: set ft=ruby :

ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip

nodes = [
  { hostname:  "pretzel",
    ip:        "172.22.22.101",
    role:      "master"
  },
  {
    hostname:  "mustard",
    ip:        "172.22.22.111",
    role:      "minion"
  },
  {
    hostname:  "nacho",
    ip:        "172.22.22.112",
    role:      "minion"
  }
]

# Shell script for basic prep
$base = <<SCRIPT
apt-get install -qqy python-software-properties
add-apt-repository -y ppa:saltstack/salt
apt-get update
SCRIPT

# Shell script for Salt minion install and config
$minion = <<SCRIPT
apt-get install -qqy salt-minion
sed -i 's/#master: salt/master: pretzel/gI' /etc/salt/minion
service salt-minion restart
SCRIPT

Vagrant.configure(2) do |config|
  if Vagrant.has_plugin?("vagrant-hostmanager")
    config.hostmanager.enabled = true
  end
  nodes.each do |node|
    config.vm.define node[:hostname] do |nodeconfig|
      nodeconfig.vm.box = "ubuntu/trusty64"
      nodeconfig.vm.hostname = node[:hostname]
      nodeconfig.vm.network :private_network, ip: node[:ip]
      nodeconfig.vm.synced_folder ".", "/vagrant", disabled: true
      nodeconfig.vm.provision :shell, inline: "echo #{ssh_pub_key} >> /root/.ssh/authorized_keys"
      nodeconfig.vm.provision :shell, inline: "echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys"
      nodeconfig.vm.provision :shell, inline: $base
      if node[:role] == "minion"
        nodeconfig.vm.provision :shell, inline: $minion
      else
        nodeconfig.vm.provision :shell, inline: "apt-get install -qqy salt-master"
      end
      nodeconfig.vm.provider :virtualbox do |v|
        v.name = node[:hostname]
        v.memory = 1024
      end
    end
  end
end
