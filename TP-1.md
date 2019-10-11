### Mon premier Vagrantfile:

```
Vagrant.configure(2) do |config|
	config.vm.box = "debian/stretch64"

	config.vm.provision "shell", inline: <<-SHELL
		apt-get update -y
		apt-get install php-geshi -y
		cd /tmp
		wget https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
		tar -xvf dokuwiki-stable.tgz
		rm /var/www/html/index.html
		cp -r dokuwiki-2018-04-22b/* /var/www/html/
		chmod -R www-data /var/www/html/
 	SHELL

 	config.vm.define "master" do |master|
		master.vm.box= "debian/stretch64"
		master.vm.network "private_network", ip: "192.168.45.11"
		master.vm.hostname = "master"
	end
  
	config.vm.define "backup" do |slave|
		slave.vm.box = "debian/stretch64"
		slave.vm.network "private_network", ip: "192.168.45.12"
		slave.vm.hostname = "backup"
	end

	config.ssh.insert_key = false
	config.ssh.private_key_path = ["Keys/id_rsa", "~/.vagrant.d/insecure_private_key"]
 	config.vm.provision "file", source: "Keys/id_rsa", destination: "~/.ssh/id_rsa"
	config.vm.provision "file", source: "Keys/id_rsa.pub", destination: "~/.ssh/authorized_keys"
	config.vm.provision "file", source: "Keys/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"

	config.vm.provision "shell", inline: <<-SHELL
		chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh/ && chmod 600 ~/.ssh/*
		ssh-keyscan 192.168.45.12 >> ~/.ssh/known_hosts
		ssh-keyscan 192.168.45.11 >> ~/.ssh/known_hosts
	SHELL
 
end
```

### Mon Deuxieme VagrantFile:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.box = "debian/stretch64"

  config.vm.provision "shell", inline: <<-SHELL
  # apt-get update
  sudo apt-get - y -qq install libapache2-mod-php7.0
  sudo apt-get - y -qq install php7.0-mbstring
  sudo useradd -m -G www-data wiki
  sudo mkdir  /opt/src
  sudo wget -q -O /opt/src/dokuwiki-stable.tgz https://download.dokuwiki.org/src/dokuwiki/dokuwiki-stable.tgz
  sudo tar zxf /opt/src/dokuwiki-stable.tgz -C /var/www/html/
  sudo chown -R wiki:www-data /var/www/html/dokuwiki-2018-04-22b/
  sudo chmod -R g+w /var/www/html/dokuwiki-2018-04-22b/
  sudo systemctl restart apache2 
  SHELL

  config.vm.define "wiki" do |wik|
  wik.vm.hostname = "wiki"
  wik.vm.network "private_network", ip: "192.168.122.132"
  end

  config.vm.define "back" do |bac|
    bac.vm.hostname = "backup"
    bac.vm.network "private_network", ip: "192.168.122.131"
    bac.vm.provision "cron", type: "shell", run: "once", inline: <<-EOF
    echo should do rsync cron
    EOF
  end

  config.vm.define "proxy" do |pro|
    pro.vm.hostname ="proxy"
    pro.vm.network "private_network", ip: "192.168.122.130"
    pro.vm.provision "init", type: "shell", run: "once", inline: <<-EOF
    apt-get install nginx
    EOF
  end 

end
```
