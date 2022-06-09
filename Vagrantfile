# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
  :log23 => {
        :box_name => "generic/rocky8",
        :ip_addr => '192.168.56.15'
  },
  :web23 => {
        :box_name => "generic/rocky8",
        :ip_addr => '192.168.56.10'
  }
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s
          box.vm.synced_folder ".", "/vagrant", disabled: true
          box.vm.network "private_network", ip: boxconfig[:ip_addr]

         box.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "1024"]            
         end
     end
  end
    config.vm.provision "ansible" do |ansible|
    #ansible.verbose = "vvv"
    ansible.playbook = "play.yml"
    #ansible.groups = {
    #  "ddns_lab" => ["ns01"],
    #  "ddns_lab" => ["client"]
    #}
    ansible.become = "true"
  end
end
