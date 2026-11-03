# -*- mode: ruby -*-
KEY_DIR = File.join(__dir__, ".ssh_keys")
Dir.mkdir(KEY_DIR) unless Dir.exist?(KEY_DIR)
unless File.exist?(File.join(KEY_DIR, "id_rsa"))
  system("ssh-keygen -t rsa -b 2048 -N '' -f #{KEY_DIR}/id_rsa -C ansible-control")
end

NODES = [
  { name: "ansible-control", host: "control-node", ip: "192.168.56.10", mem: 512 },
  { name: "web1",            host: "webserver-01", ip: "192.168.56.11", mem: 384 },
  { name: "web2",            host: "webserver-02", ip: "192.168.56.12", mem: 384 },
  { name: "db1",             host: "dbserver-01",  ip: "192.168.56.20", mem: 768 },
]

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"

  NODES.each do |n|
    config.vm.define n[:name] do |node|
      node.vm.hostname = n[:host]
      node.vm.network "private_network", ip: n[:ip]

      node.vm.provider "virtualbox" do |vb|
        vb.name   = n[:name]
        vb.memory = n[:mem]
        vb.cpus   = 1
      end

      if n[:name] == "ansible-control"
        node.vm.provision "shell", inline: <<-'SHELL'
          export DEBIAN_FRONTEND=noninteractive
          apt-get update -y
          apt-get install -y ansible
          install -d -m 700 -o vagrant -g vagrant /home/vagrant/.ssh
          install -m 600 -o vagrant -g vagrant /vagrant/.ssh_keys/id_rsa     /home/vagrant/.ssh/id_rsa
          install -m 644 -o vagrant -g vagrant /vagrant/.ssh_keys/id_rsa.pub /home/vagrant/.ssh/id_rsa.pub
          printf "Host 192.168.56.*\n  StrictHostKeyChecking no\n  UserKnownHostsFile /dev/null\n" > /home/vagrant/.ssh/config
          chown vagrant:vagrant /home/vagrant/.ssh/config
          chmod 600 /home/vagrant/.ssh/config
        SHELL
      else
        node.vm.provision "shell", inline: <<-'SHELL'
          PUB="$(cat /vagrant/.ssh_keys/id_rsa.pub)"
          grep -qxF "$PUB" /home/vagrant/.ssh/authorized_keys || echo "$PUB" >> /home/vagrant/.ssh/authorized_keys
        SHELL
      end
    end
  end
end
