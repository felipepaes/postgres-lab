# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

require 'io/console'

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_version = "1905.1"

  config.vm.network :forwarded_port, host: 5432, guest: 5432
  config.vm.network :forwarded_port, host: 5050, guest: 5050
  config.vm.network :forwarded_port, host: 8888, guest: 8888 
  # config.vm.network "private_network", ip: "192.168.33.10"

  # config.vm.synced_folder "../data", "/vagrant_data"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
  end

  header = "\n\
  ██████╗  ██████╗ ███████╗████████╗ ██████╗ ██████╗ ███████╗███████╗
  ██╔══██╗██╔═══██╗██╔════╝╚══██╔══╝██╔════╝ ██╔══██╗██╔════╝██╔════╝
  ██████╔╝██║   ██║███████╗   ██║   ██║  ███╗██████╔╝█████╗  ███████╗
  ██╔═══╝ ██║   ██║╚════██║   ██║   ██║   ██║██╔══██╗██╔══╝  ╚════██║
  ██║     ╚██████╔╝███████║   ██║   ╚██████╔╝██║  ██║███████╗███████║
  ╚═╝      ╚═════╝ ╚══════╝   ╚═╝    ╚═════╝ ╚═╝  ╚═╝╚══════╝╚══════╝
                                                                     
  ██╗      █████╗ ██████╗                                            
  ██║     ██╔══██╗██╔══██╗                                           
  ██║     ███████║██████╔╝                                           
  ██║     ██╔══██║██╔══██╗                                           
  ███████╗██║  ██║██████╔╝                                           
  ╚══════╝╚═╝  ╚═╝╚═════╝                                            
  \n"

 

  if ARGV[0] == "up" || ARGV[0] == "provision"
    print header
    puts "Welcome to PostgresLab!"
    print "Let's configure your lab.\n\n"

    print "postgresql username: "
    database_username = STDIN.gets.chomp
    print "postgresql password: "
    database_password = STDIN.noecho(&:gets).chomp
    print "\ndatabase name: "
    database_name = STDIN.gets.chomp

    config.vm.post_up_message = "\n
    -- Your PostgreSQL credentials --
    user: #{database_username}
    password: #{database_password}
    database: #{database_name}
    host: localhost
    port: 5432
    \n"
  end


 
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  if ARGV[0] == "up" || ARGV[0] == "provision"
      config.vm.provision "shell", inline: <<-SHELL

        # yum update -y
        yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
        yum install -y postgresql12
        yum install -y postgresql12-server
        
        # init postgres service
        /usr/pgsql-12/bin/postgresql-12-setup initdb
        systemctl enable postgresql-12
        systemctl start postgresql-12
    
        # configure postgres network and auth protocol
        sed -i "s/#listen_addresses = 'localhost'/listen_addresses = '*'/" "/var/lib/pgsql/12/data/postgresql.conf"
        echo "host    all             all             all                     md5" >> "/var/lib/pgsql/12/data/pg_hba.conf"
        echo "client_encoding = utf8" >> "/var/lib/pgsql/12/data/postgresql.conf"
        sed -i "s/::1\/128                 ident/::1\/128                 md5/" "/var/lib/pgsql/12/data/pg_hba.conf" 
      
        # restart postgres service:
        systemctl restart  postgresql-12
    
        # create postgres user and database
        sudo -u postgres psql -c "CREATE USER #{database_username} WITH ENCRYPTED PASSWORD '#{database_password}' LOGIN CREATEDB;"
        sudo -u postgres psql -c "CREATE DATABASE #{database_name} WITH OWNER #{database_username};"

      SHELL
    end
end
