# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure (2) do |config|

  config.vm.define "DynamicWeb" do |vmconfig|
    vmconfig.vm.box = "ubuntu/jammy64"
    vmconfig.vm.hostname = "DynamicWeb"

    vmconfig.vm.network "forwarded_port", guest: 8083, host: 8083
    vmconfig.vm.network "forwarded_port", guest: 8081, host: 8081
    vmconfig.vm.network "forwarded_port", guest: 8082, host: 8082
    vmconfig.vm.network "private_network", ip: "192.168.56.10"

    vmconfig.vm.provider "virtualbox" do |vbx|
      vbx.memory = "2048"
      vbx.cpus = "2"
      vbx.customize ["modifyvm", :id, "--audio", "none"]
    end

    vmconfig.vm.provision "shell", inline: <<-SHELL
      sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
      sudo systemctl restart sshd
    SHELL
  end

end
