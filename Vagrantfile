# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # All Vagrant configuration is done here. The most common configuration
  # options are documented and commented below. For a complete reference,
  # please see the online documentation at vagrantup.com.

  # config.vm.box = "hashicorp/precise32"
  config.vm.box = "ubuntu/trusty32"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  #
  # config.vm.network "forwarded_port", guest: 5672, host: 5555 # rabbitMQ
  config.vm.network "forwarded_port", guest: 15672, host: 5555 # rabbitMQ web UI
  # config.vm.network "forwarded_port", guest: 6379, host: 6666 # Redis

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # If true, then any SSH connections made will enable agent forwarding.
  # Default value: false
  # config.ssh.forward_agent = true

  config.vm.synced_folder "routemaster", "/routemaster"
  # config.vm.synced_folder "jukebox", "/jukebox"
  # config.vm.synced_folder "jukestats", "/jukestats"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  #   # Don't boot with headless mode
    # vb.gui = true
  #
  #   # Use VBoxManage to customize the VM. For example to change memory:
  #   vb.customize ["modifyvm", :id, "--memory", "1024"]
  end
  #
  # View the documentation for the provider you're using for more
  # information on available options.

  # NEXT STEP VM NEEDS TO ACCEPT HTTP (80) AND CONVERT IT TO HTTPS BEFORE PROXYING
  # TO ROUTEMASTER (5672) (ngnix or apache or tunnels). Port 80 (guest) then
  # needs port forwarding by vagrant to (56722) on the host, this is where the
  # client will connect.

  # sudo apt-get -y upgrade
  $script = <<-SCRIPT
  sudo echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
  curl http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -
  apt-get update
  sudo apt-get install rabbitmq-server -y
  sudo rabbitmq-plugins enable rabbitmq_management

  sudo rabbitmqctl add_vhost routemaster.development

  # TODO: upload a rabbitmq.config which disables loopback_users
  sudo rabbitmqctl set_permissions -p routemaster.development guest ".*" ".*" ".*"

  # new user because guest is a loopback_user so can only login via localhost
  sudo rabbitmqctl add_user admin admin
  sudo rabbitmqctl set_permissions -p routemaster.development admin ".*" ".*" ".*"
  sudo rabbitmqctl set_user_tags admin administrator

  sudo rabbit-server restart
  SCRIPT

  config.vm.provision "shell", inline: $script
end
