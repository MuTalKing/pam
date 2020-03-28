

Vagrant.configure("2") do |config|

config.vm.provision "shell", path: "install.sh"

  config.vm.define "otuslinux" do |otuslinux|

    otuslinux.vm.box = "centos/7"

    otuslinux.vm.network "private_network", ip: "192.168.10.120"

    otuslinux.vm.hostname = "otuslinux"

    otuslinux.vm.provider :virtualbox do |vb|

      vb.customize ["modifyvm", :id, "--memory", "512"]

      vb.customize ["modifyvm", :id, "--cpus", "2"]

      end 

    otuslinux.vm.provision "shell", inline: <<-SHELL

	mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh

	
    SHELL
    
    end
  
end
