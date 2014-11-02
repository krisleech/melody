# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # config.vm.box = "hashicorp/precise32"
  config.vm.box = "ubuntu/trusty64"

  # Port forwarding
  #
  # config.vm.network "forwarded_port", guest: 5672, host: 5555 # rabbitMQ
  # config.vm.network "forwarded_port", guest: 6379, host: 6666 # Redis
  
  config.vm.network "forwarded_port", guest: 15672, host: 5555 # RabbitMQ web UI
  config.vm.network "forwarded_port", guest: 443,   host: 4434 # Routemaster HTTP

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

  config.vm.provider "virtualbox" do |vb|
  end

  # sudo apt-get -y upgrade
  $script = <<-SCRIPT
  sudo apt-get install build-essential -y -qq
  sudo apt-get install libssl-dev -y -qq
  sudo apt-get install git -y -qq

  cd

  # Ruby 2.1.4
  # sudo add-apt-repository ppa:brightbox/ruby-ng
  # sudo apt-get update
  # sudo apt-get install ruby2.1-dev ruby2.1 -y -qq

  # Ruby 2.1.2
  # uses packager.io distribution
  curl https://s3.amazonaws.com/pkgr-buildpack-ruby/current/ubuntu-14.04/ruby-2.1.2.tgz -o - | sudo tar xzf - -C /usr/local
  echo "gem: --no-ri --no-rdoc" > ~/.gemrc

  # Redis
  sudo apt-get install redis-server -y -qq

  # RabbitMQ
  sudo echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list
  curl http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -
  apt-get update
  sudo apt-get install rabbitmq-server -y -qq
  sudo rabbitmq-plugins enable rabbitmq_management

  # Configure RabbitMQ
  sudo rabbitmqctl add_vhost routemaster.development
  sudo rabbitmqctl set_permissions -p routemaster.development guest ".*" ".*" ".*"
  sudo echo '[{rabbit, [{loopback_users, []}]}].' >> /etc/rabbitmq/rabbitmq.config

  sudo service rabbitmq-server restart

  # Install tunnels (SSL proxy)
  sudo gem install tunnels
  # FIXME: use upstart for tunnels so it starts on reboot
  sudo tunnels 443 17890 &

  # Install Routemaster
  sudo gem install bundler
  cd /routemaster
  bundle install --quiet
  sudo foreman export upstart /etc/init --app routemaster --user vagrant
  sudo service routemaster start

  # echo `sudo netstat -tulpn | grep LISTEN`
  SCRIPT

  config.vm.provision "shell", inline: $script
end
