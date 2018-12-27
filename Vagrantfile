Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
   config.vm.provision "shell", inline: <<-SHELL
     apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 9DA31620334BD75D9DCB49F368818C72E52529D4 
     echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.0.list
     apt-get update
     apt-get install -y mongodb-org
     cd /home/vagrant && openssl rand -base64 741 > mongodb-keyfile && chmod 600 mongodb-keyfile
   SHELL

  config.vm.define "db1" do |db1|
    db1.vm.network "private_network", ip: "192.168.33.10"
    db1.vm.synced_folder "mongodb_config.conf", "/etc/mongodb_config.conf"
    db1.vm.synced_folder "mongos.conf", "/etc/mongos.conf"
  end

  config.vm.define "db2" do |db2|
    db2.vm.network "private_network", ip: "192.168.33.20"
    db2.vm.synced_folder "mongodb_shard.conf", "/etc/mongodb_shard.conf"
  end

  config.vm.define "db3" do |db3|
    db3.vm.network "private_network", ip: "192.168.33.30"
    db3.vm.synced_folder "mongodb_shard.conf", "/etc/mongodb_shard.conf"
  end
end
