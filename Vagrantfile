Vagrant.configure("2") do |config|
    config.vm.box = "centos/8"
    config.vm.provider "virtualbox" do |vb|
      vb.memory = "2048"
    end
    config.vm.network "private_network", ip: "192.168.33.10"
    config.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook.yaml"
      ansible.become = true
    end
  end
  