# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :Router1 => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 10022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "r1-2"},
                   {adapter: 3, virtualbox__intnet: "r1-3"},
                   {adapter: 4, virtualbox__intnet: "r1-lan"},
                ]
  },
  :Router2 => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 11022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "r1-2"},
                   {adapter: 3, virtualbox__intnet: "r2-3"},
                   {adapter: 4, virtualbox__intnet: "r2-lan"},
                ]
  },
  :Router3 => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 12022},
        :net => [
                   {adapter: 2, virtualbox__intnet: "r1-3"},
                   {adapter: 3, virtualbox__intnet: "r2-3"},
                   {adapter: 4, virtualbox__intnet: "r3-lan"},
                ]
  },
  :provision => {
        :box_name => "centos/7",
        :forward => {:guest => 22, :host => 13022},
        :net => [
                   {adapter: 2, ip: '192.168.220.2', netmask: "255.255.255.0"},
                ]
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s
        box.vm.network "forwarded_port", boxconfig[:forward]

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
          sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
          systemctl restart sshd
        SHELL
        case boxname.to_s
        when "provision"
          box.vm.provision "shell", run: "always", inline: <<-SHELL
            yum install epel-release -y
            yum install ansible -y
            cp -r /vagrant/* /root/
            cd /root
            ansible-playbook ospf.yml
            SHELL
        end        
    end
  end
end
