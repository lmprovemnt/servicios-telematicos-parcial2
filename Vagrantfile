Vagrant.configure("2") do |config|

  config.vm.box = "bento/ubuntu-24.04"
  config.vm.box_version = "202510.26.0"

  # =========================
  # SERVIDOR 1 - FIREWALL/NAT
  # =========================
  config.vm.define "srv1" do |srv1|
    srv1.vm.hostname = "srv1"

    # Red privada hacia srv2
    srv1.vm.network "private_network",
      ip: "192.168.50.1",
      virtualbox__intnet: "red_srv2"

    # Red privada hacia el cliente
    srv1.vm.network "private_network",
      ip: "192.168.60.1",
      virtualbox__intnet: "red_cliente"

    srv1.vm.provider "virtualbox" do |vb|
      vb.name = "parcial-srv1"
      vb.memory = 1024
      vb.cpus = 1
    end
  end

  # =========================
  # SERVIDOR 2 - FTPS + SFTP
  # =========================
  config.vm.define "srv2" do |srv2|
    srv2.vm.hostname = "srv2"

    # Solo comparte esta red con srv1
    srv2.vm.network "private_network",
      ip: "192.168.50.2",
      virtualbox__intnet: "red_srv2"

    srv2.vm.provider "virtualbox" do |vb|
      vb.name = "parcial-srv2"
      vb.memory = 2048
      vb.cpus = 2
    end
  end

  # =========================
  # CLIENTE
  # =========================
  config.vm.define "client" do |client|
    client.vm.hostname = "client"

    # Solo comparte esta red con srv1
    client.vm.network "private_network",
      ip: "192.168.60.20",
      virtualbox__intnet: "red_cliente"

    client.vm.provider "virtualbox" do |vb|
      vb.name = "parcial-client"
      vb.memory = 2048
      vb.cpus = 2
    end
  end

end
