Vagrant.configure("2") do |config|
  (1..2).each do |i|
    config.vm.define "cnode#{i}" do |node|
      node.vm.box = "bento/ubuntu-14.04"
      node.vm.hostname = "cnode#{i}"
      node.vm.network "private_network", ip: "192.168.1.1#{i}"

      node.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.name = "cnode#{i}"
        vb.memory = "1024"
      end
    end
  end
end
