# -*- mode: ruby -*-
# vi: set ft=ruby :
Vagrant.configure(2) do |config|
  config.vm.define "srv1" do |srv1|
    config.vm.provider "virtualbox" do |srv1|
      srv1.name = "srv1"
    end
    srv1.vm.box = "minimal/xenial64"
        srv1.vm.network "private_network", virtualbox__intnet: "srv1_leaf1", auto_config: false
        srv1.vm.provision "shell", inline: <<-SHELL
          sudo ip link set enp0s8 up          
          sudo apt-get update
          sudo apt-get -y install iperf3
          sudo apt-get -y install traceroute
          sudo apt-get -y install tcpdump
          sudo apt-get -y install lldpd
          sudo service lldpd start
          sudo ip addr add 192.168.11.111/24 dev enp0s8
          sudo ip route add 192.168.22.0/24 via 192.168.11.254          
          sudo ip link set enp0s8 up
        SHELL
        srv1.vm.hostname = "srv1"
  end
  config.vm.define "srv2" do |srv2|
    config.vm.provider "virtualbox" do |srv2|
      srv2.name = "srv2"
    end
    srv2.vm.box = "minimal/xenial64"
        srv2.vm.network "private_network", virtualbox__intnet: "srv2_leaf2", auto_config: false
        srv2.vm.provision "shell", inline: <<-SHELL
          sudo ip addr add 192.168.11.222/24 dev enp0s8
          sudo ip route add 192.168.22.0/24 via 192.168.11.254
          sudo ip link set enp0s8 up
          sudo apt-get -y install iperf3
          sudo apt-get update
          sudo apt-get -y install traceroute
          sudo apt-get -y install tcpdump
          sudo apt-get -y install lldpd
          sudo service lldpd start

        SHELL
        srv2.vm.hostname = "srv2"
  end
  config.vm.define "srv3" do |srv3|
    config.vm.provider "virtualbox" do |srv3|
      srv3.name = "srv3"
    end
    srv3.vm.box = "minimal/xenial64"
        srv3.vm.network "private_network", virtualbox__intnet: "srv3_leaf2", auto_config: false
        srv3.vm.provision "shell", inline: <<-SHELL
          sudo ip addr add 192.168.22.222/24 dev enp0s8
          sudo ip link set enp0s8 up
          sudo ip route add 192.168.11.0/24 via 192.168.22.254
          sudo apt-get -y install iperf3
          sudo apt-get update
          sudo apt-get -y install traceroute
          sudo apt-get -y install tcpdump
          sudo apt-get -y install lldpd
          sudo service lldpd start
        SHELL
        srv3.vm.hostname = "srv3"
  end
  
  config.vm.define "leaf1" do |leaf1|
    leaf1.ssh.insert_key = false
    leaf1.vm.box = "juniper/vqfx10k-re"
    leaf1.vm.synced_folder '.', '/vagrant', disabled: true
    leaf1.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf1_internal"
    leaf1.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf1_reserved"
    leaf1.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf1_spine"
    leaf1.vm.network "private_network", auto_config: false, virtualbox__intnet: "srv1_leaf1"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "leaf1"
      vb.customize ['modifyvm',:id,'--memory','1024']
    end
    leaf1.vm.provision "ansible" do |ansible|
      ansible.playbook = "setup_evpn_vxlan.yml"
    end
  end
  config.vm.define "leaf1-pfe" do |pfe|
    pfe.ssh.insert_key = false
    pfe.vm.box = "juniper/vqfx10k-pfe"
    pfe.vm.synced_folder '.', '/vagrant', disabled: true
    pfe.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf1_internal"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "leaf1-pfe"
      vb.customize ['modifyvm',:id,'--memory','2048']
      vb.customize ['modifyvm',:id,'--cpuexecutioncap','50']
    end
  end

  config.vm.define "spine" do |spine|
    spine.ssh.insert_key = false
    spine.vm.box = "juniper/vqfx10k-re"
    spine.vm.synced_folder '.', '/vagrant', disabled: true
    spine.vm.network "private_network", auto_config: false, virtualbox__intnet: "spine_internal"
    spine.vm.network "private_network", auto_config: false, virtualbox__intnet: "spine_reserved"
    spine.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf1_spine"
    spine.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf2_spine"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "jnpr-spine"
    end
    spine.vm.provision "ansible" do |ansible|
      ansible.playbook = "setup_evpn_vxlan.yml"
    end
  end
  config.vm.define "spine-pfe" do |pfe|
    pfe.ssh.insert_key = false
    pfe.vm.box = "juniper/vqfx10k-pfe"
    pfe.vm.synced_folder '.', '/vagrant', disabled: true
    pfe.vm.network "private_network", auto_config: false, virtualbox__intnet: "spine_internal"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "spine-pfe"
      vb.customize ['modifyvm',:id,'--memory','2048']
      vb.customize ['modifyvm',:id,'--cpuexecutioncap','50']
    end
  end
  config.vm.define "leaf2" do |leaf2|
    leaf2.ssh.insert_key = false
    leaf2.vm.box = "juniper/vqfx10k-re"
    leaf2.vm.synced_folder '.', '/vagrant', disabled: true
    leaf2.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf2_internal"
    leaf2.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf2_reserved"
    leaf2.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf2_spine"
    leaf2.vm.network "private_network", auto_config: false, virtualbox__intnet: "srv2_leaf2"
    leaf2.vm.network "private_network", auto_config: false, virtualbox__intnet: "srv3_leaf2"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "leaf2"
      vb.customize ['modifyvm',:id,'--memory','1024']
    end
    leaf2.vm.provision "ansible" do |ansible|
      ansible.playbook = "setup_evpn_vxlan.yml"
    end
  end
  config.vm.define "leaf2-pfe" do |pfe|
    pfe.ssh.insert_key = false
    pfe.vm.box = "juniper/vqfx10k-pfe"
    pfe.vm.synced_folder '.', '/vagrant', disabled: true
    pfe.vm.network "private_network", auto_config: false, virtualbox__intnet: "leaf2_internal"
    config.vm.provider "virtualbox" do |vb|
      vb.name = "leaf2-pfe"
      vb.customize ['modifyvm',:id,'--memory','2048']
      vb.customize ['modifyvm',:id,'--cpuexecutioncap','50']
    end
  end
#  config.vm.provision "ansible" do |ansible|
#         ansible.groups = {
#             "leaf" => ["leaf1a", "leaf1b", "leaf2" ],
#             "spine"  => ["spine"],
#             "all:children" => ["leaf", "spine"]
#         }
#         ansible.playbook = "setup_evpn_vxlan.yml"
#  end

end
