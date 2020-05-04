# -*- mode: ruby -*-
# vi: set ft=ruby :


require 'io/console'

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.box_version = "1905.1"
  config.vm.define 'postgresLab'
  config.vm.hostname = 'postgresLab'
  config.vbguest.auto_update = false

  config.vm.network :forwarded_port, host: 5432, guest: 5432
  config.vm.network :forwarded_port, host: 5050, guest: 5050
  # config.vm.network :forwarded_port, host: 8888, guest: 8888 
  # config.vm.network "private_network", ip: "192.168.33.10"
  # config.vm.synced_folder "dependencies", "/home/vagrant/dependencies"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
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
    Thank you for using postgresLab!\n
    -- Your PostgreSQL credentials --
    user: #{database_username}
    password: #{database_password}
    database: #{database_name}
    host: localhost
    port: 5432\n
    -- pgAdmin4 --
    Access pgAdmin4 at: http:localhost:5050"
  end


 
  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  if ARGV[0] == "up" || ARGV[0] == "provision"
      config.vm.provision "shell", inline: <<-SHELL

        # install development tools
        yum -y groupinstall 'Development Tools'

        # install postgresql
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
        sed -i "s~::1\/128                 ident~::1\/128                 md5~" "/var/lib/pgsql/12/data/pg_hba.conf"
        
        # restart postgres service:
        systemctl restart  postgresql-12
        
        # create postgres user and database
        sudo -u postgres psql -c "CREATE USER #{database_username} WITH ENCRYPTED PASSWORD '#{database_password}' LOGIN CREATEDB;"
        sudo -u postgres psql -c "CREATE DATABASE #{database_name} WITH OWNER #{database_username};"
        
        # pgAdmin4
        echo "INSTALLING pgAdmin4"
        yum -y install python3 python3-devel
        python3 -m venv pgadmin4
        venv=/home/vagrant/pgadmin4/bin/python
        $venv -m pip install pip --upgrade
        $venv -m pip install wheel
        curl -# -O https://ftp.postgresql.org/pub/pgadmin/pgadmin4/v4.21/pip/pgadmin4-4.21-py2.py3-none-any.whl
        $venv -m pip install pgadmin4-4.21-py2.py3-none-any.whl

        # config pgadmin
        echo "DEFAULT_SERVER = '0.0.0.0'" >> /home/vagrant/pgadmin4/lib/python3.6/site-packages/pgadmin4/config_local.py
        echo "SERVER_MODE = False" >> /home/vagrant/pgadmin4/lib/python3.6/site-packages/pgadmin4/config_local.py
        echo "MASTER_PASSWORD_REQUIRED = False" >> /home/vagrant/pgadmin4/lib/python3.6/site-packages/pgadmin4/config_local.py
        
        echo "[Unit]" >> /etc/systemd/system/pgadmin4.service
        echo "Description=Pgadmin4 Service" >> /etc/systemd/system/pgadmin4.service
        echo "After=network.target\n" >> /etc/systemd/system/pgadmin4.service
                
        echo "[Service]" >> /etc/systemd/system/pgadmin4.service
        echo "WorkingDirectory=/home/vagrant/pgadmin4/" >> /etc/systemd/system/pgadmin4.service
        echo 'Environment="PATH=/home/vagrant/pgadmin4/bin\"' >> /etc/systemd/system/pgadmin4.service
        echo "ExecStart=/home/vagrant/pgadmin4/bin/python /home/vagrant/pgadmin4/lib/python3.6/site-packages/pgadmin4/pgAdmin4.py" >> /etc/systemd/system/pgadmin4.service
        echo "PrivateTmp=true\n" >> /etc/systemd/system/pgadmin4.service
                
        echo "[Install]" >> /etc/systemd/system/pgadmin4.service
        echo "WantedBy=multi-user.target" >> /etc/systemd/system/pgadmin4.service
        
        # activate pgadmin4 service
        systemctl start pgadmin4
        systemctl enable pgadmin4
      SHELL
    end
end
