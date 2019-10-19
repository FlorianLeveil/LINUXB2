---


---

<h1 id="tp-apache">TP APACHE</h1>
<p>Vagranfile:</p>
<pre><code># -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
	config.vm.box = "debian/stretch64"
	config.vm.box_check_update = false

config.vm.define "master" do |master|
	master.vm.hostname = "wikileaks"
	master.vm.network "private_network", ip: "192.168.122.131"
	master.vm.network "forwarded_port", guest: 80, host: 8181
	config.vm.provision "shell", inline: &lt;&lt;-SHELL
		  apt-get update
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
	master.vm.provision "file", source: "files/default-conf.conf", destination: "/etc/apache2/sites-available/default-conf.conf"
	master.vm.provision "file", source: "files/index.html", destination: "/var/www/html/index.html"
	master.vm.provision "file", source: "files/.htaccess", destination: "/var/www/html/.htaccess"
	master.vm.provision "file", source: "files/wiki-conf.conf", destination: "/etc/apache2/sites-available/wiki-conf.conf"

end

config.vm.define "back" do |back|
	back.vm.hostname = "backos"
	back.vm.network "private_network", ip: "192.168.122.132"
	back.vm.network "forwarded_port", guest: 80, host: 8181
	config.vm.provision "shell", inline: &lt;&lt;-SHELL
		  apt-get update
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
	back.vm.provision "file", source: "files/default-conf.conf", destination: "/etc/apache2/sites-available/default-conf.conf"
	back.vm.provision "file", source: "files/index.html", destination: "/var/www/html/index.html"
	back.vm.provision "file", source: "files/.htaccess", destination: "/var/www/html/.htaccess"
	back.vm.provision "file", source: "files/wiki-conf.conf", destination: "/etc/apache2/sites-available/wiki-conf.conf"

	end
end
</code></pre>
<p>File1 (index.html):</p>
<pre><code>&lt;!DOCTYPE html&gt;
&lt;html lang="fr"&gt;
&lt;head&gt;
&lt;meta charset="UTF-8"&gt;
&lt;title&gt;idk&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;p style="text-align: center"&gt;Ce serveur n'est pas accessible.&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>
<p>File2 (.hatacess):</p>
<pre><code>Options -Indexes
Errordocument 404 index.html
DirectoryIndex index.html
Require all granted
</code></pre>
<p>File3(default-conf.conf)</p>
<pre><code>&lt;VirtualHost *:80&gt;

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/

	&lt;Directory "/var/www/html"&gt;
		AllowOverride All
	&lt;/Directory&gt;

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

&lt;/VirtualHost&gt;
</code></pre>
<p>File4(wiki-conf.conf)</p>
<pre><code>&lt;VirtualHost *:80&gt;
	ServerName idk
	DocumentRoot "/var/www/dokuwiki"

	&lt;Directory "/var/www/dokuwiki"&gt;
		Options +FollowSymLinks
		AllowOverride all
		Require all granted
	&lt;/Directory&gt;

	ErrorLog /var/log/apache2/error.example.com.log
	CustomLog /var/log/apache2/access.example.com.log combined

&lt;/VirtualHost&gt;
</code></pre>

