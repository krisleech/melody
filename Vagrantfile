# -*- mode: ruby -*-
# vi: set ft=ruby :

# TODO: get IP addresses from ENV

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  config.vm.define "jukebox" do |jb|
    jb.vm.network "private_network", ip: "192.168.0.200"

    jb.vm.synced_folder "jukebox", "/jukebox"

    jb.vm.provider "virtualbox" do |vb|
    end

    $script = <<-SCRIPT
    sudo apt-get install build-essential -y -qq
    sudo apt-get install git -y -qq
    sudo apt-get install sqlite3 libsqlite3-dev -y -qq

    # Ruby 2.1.4
    sudo add-apt-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.1-dev ruby2.1 -y -qq
    echo "gem: --no-ri --no-rdoc" > ~/.gemrc

    # Install Tunnels (SSL proxy)
    sudo gem install tunnels
    sudo echo -e 'start on startup\nexec sudo tunnels 0.0.0.0:443 0.0.0.0:3000' >> /etc/init/tunnels.conf
    sudo service tunnels start

    # Install App
    sudo gem install bundler
    cd /jukebox
    echo 'bundle installing...'
    bundle install --quiet
    sudo foreman export upstart /etc/init --app jukebox --user vagrant
    sudo service jukebox start
    bundle exec rake db:migrate db:seed
    SCRIPT

    jb.vm.provision "shell", inline: $script
  end

  config.vm.define "routemaster" do |rm|
    rm.vm.network "private_network", ip: "192.168.0.201"

    # rm.ssh.forward_agent = true

    rm.vm.synced_folder "routemaster", "/routemaster"

    rm.vm.provider "virtualbox" do |vb|
    end

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
    sudo apt-get update
    sudo apt-get install rabbitmq-server -y -qq
    sudo rabbitmq-plugins enable rabbitmq_management

    # Configure  RabbitMQ
    sudo rabbitmqctl add_vhost routemaster.development
    sudo rabbitmqctl set_permissions -p routemaster.development guest ".*" ".*" ".*"
    sudo echo '[{rabbit, [{loopback_users, []}]}].' >> /etc/rabbitmq/rabbitmq.rm

    sudo service rabbitmq-server restart

    # Install Tunnels (SSL proxy)
    sudo gem install tunnels
    sudo echo -e 'start on startup\nexec sudo tunnels 0.0.0.0:443 0.0.0.0:17890' >> /etc/init/tunnels.conf
    sudo service tunnels start

    # Install Routemaster
    sudo gem install bundler
    cd /routemaster
    echo 'bundle installing...'
    bundle install --quiet
    sudo foreman export upstart /etc/init --app routemaster --user vagrant
    sudo service routemaster start
    SCRIPT

    rm.vm.provision "shell", inline: $script
  end

  config.vm.define "jukestats" do |js|
    js.vm.network "private_network", ip: "192.168.0.202"

    js.vm.synced_folder "jukestats", "/jukestats"

    js.vm.provider "virtualbox" do |vb|
    end

    $script = <<-SCRIPT
    sudo apt-get install build-essential -y -qq
    sudo apt-get install git -y -qq
    sudo apt-get install sqlite3 libsqlite3-dev -y -qq

    cd

    # Ruby 2.1.4
    sudo add-apt-repository ppa:brightbox/ruby-ng
    sudo apt-get update
    sudo apt-get install ruby2.1-dev ruby2.1 -y -qq
    echo "gem: --no-ri --no-rdoc" > ~/.gemrc

    # Install Tunnels (SSL proxy)
    sudo gem install tunnels
    sudo echo -e 'start on startup\nexec sudo tunnels 0.0.0.0:443 0.0.0.0:3000' >> /etc/init/tunnels.conf
    sudo service tunnels start

    # Install App
    sudo gem install bundler
    cd /jukestats
    echo 'bundle installing...'
    bundle install --quiet
    sudo foreman export upstart /etc/init --app jukestats --user vagrant
    sudo service jukestats start
    bundle exec rake db:migrate
    SCRIPT

    js.vm.provision "shell", inline: $script
  end
end
