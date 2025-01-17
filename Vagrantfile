# -*- mode: ruby -*-
# vi: set ft=ruby :
BOX_SERVER  = "ubuntu/focal64"
BOX_DESKTOP = "gusztavvargadr/ubuntu-desktop"
VBNET = "bind9" #virtualbox__intnet: VBNET
VBFOLDER    = "/DNSLAB9"

DOMAIN = "aula104.local"
RED    = "192.168.1"
DNSIP  = "#{RED}.2"

$dnsclient = <<-SHELL
  # apt-get update
  echo "nameserver $1\ndomain $2">/etc/resolv.conf
SHELL

$apacheserver = <<-SHELL
  apt-get update
  apt-get install -y apache2
  echo "<h1>Bienvenido a $1! ($2)</h>">/var/www/html/index.html
SHELL

$nginxserver = <<-SHELL
  apt-get update
  apt-get install -y nginx
  echo "<h1>Bienvenido a $1! ($2)</h>">/var/www/html/index.nginx-debian.html
SHELL

services = {
  "nginx"   => { :ip => "#{RED}.10", :provision=>$nginxserver,  :port=> "8080" },
  "apache1" => { :ip => "#{RED}.11", :provision=>$apacheserver, :port=> "8081" },
  "apache2" => { :ip => "#{RED}.12", :provision=>$apacheserver, :port=> "8082" },
}

Vagrant.configure("2") do |config|
  # config general
  # config.vm.box = BOX_SERVER
  # if Vagrant.has_plugin?("vagrant-vbguest")
  #  config.vbguest.auto_update = false
  # end

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 1
    vb.memory = 1024
    vb.customize ["modifyvm", :id, "--groups", VBFOLDER]
  end

  # dns 
  config.vm.define :dns do |guest|
    guest.vm.provider "virtualbox" do |vb, subconfig|
      vb.name = "dns"
      subconfig.vm.hostname = "dns.#{DOMAIN}"
      subconfig.vm.network :private_network, ip: DNSIP,  virtualbox__intnet: VBNET 
    end
    guest.vm.provision "shell", name: "dns-server", path: "enable-bind9.sh", args: "#{DNSIP} #{DOMAIN}"
  end

  # services 
  services.each_with_index do |(hostname, info), idx|
    config.vm.define hostname do |guest|
      guest.vm.provider :virtualbox do |vb, subconfig|
        vb.name = hostname
        subconfig.vm.hostname = "#{hostname}.#{DOMAIN}"
        subconfig.vm.network :private_network, ip: info[:ip], virtualbox__intnet: VBNET
      end
      guest.vm.provision "shell", name: "dns-client \##{idx}", inline: $dnsclient, args: "#{DNSIP} #{DOMAIN}"
      guest.vm.provision "shell", name: "#{hostname}:#{info[:port]}", inline: info[:provision], args:  "#{hostname} #{DOMAIN}"
      guest.vm.network "forwarded_port", guest: 80, host: info[:port]
    end 
  end
  
  # clientes GUI
  (1..2).each do |id|
    config.vm.define "client#{id}" do |guest|
      guest.vm.provider "virtualbox" do |vb, subconfig|
        vb.name = "client#{id}"
        if id>1
          vb.gui = true
          vb.cpus = 2
          vb.memory = 2048
          subconfig.vm.box = BOX_DESKTOP
          subconfig.vbguest.auto_update = true
        end
        subconfig.vm.hostname = "client#{id}.#{DOMAIN}"
        subconfig.vm.network :private_network, ip: "#{RED}.#{100+id}",  virtualbox__intnet: VBNET
        # subconfig.vm.network :private_network, type: "dhcp", ip: "#{RED}.1"
      end
      guest.vm.provision "shell", name: "dns-client", inline: $dnsclient, args: "#{DNSIP} #{DOMAIN}"
      guest.vm.provision "shell", name: "testing dns", inline: <<-SHELL
        dig google.com +short               # valida los forwaders
        dig -x 192.168.1.2 +short           # valida la resolución inversa
        ping -a -c 1 apache1                # valida la resolucion no FQDN
        ping -a -c 1 apache2.aula104.local  # valida la resolución FQDN
        curl apache1 --no-progress-meter    # valida apache1 instalado
        curl apache2 --no-progress-meter    # valida apache2 instalado
        curl nginx --no-progress-meter      # valida nginx instalado
        nslookup nginx                      # valida la resolucion con nslookup (-debug)
      SHELL
    end
  end
end

  # clients DHCP
  (1..1).each do |id|
    config.vm.define "client#{id}" do |guest|
      guest.vm.provider "virtualbox" do |vb, subconfig|
        vb.name = "client#{id}"
        subconfig.vm.hostname = "client#{id}.#{DOMAIN}"

        subconfig.vm.network :private_network, ip: "192.168.33.#{150+id}",  virtualboxintnet: LAB
      end
      guest.vm.provision "shell", name: "dns-client", inline: $dnsclient, args: DNSIP
      guest.vm.provision "shell", name: "testing", inline: <<-SHELL
        dig google.com +short
        dig -x 192.168.33.10 +short
        ping -a -c 1 apache1
        ping -a -c 1 apache2.ZONA.COM.
        # curl apache1 --no-progress-meter 
        # curl apache2 --no-progress-meter 
        # curl nginx --no-progress-meter 
        ping -a -c 1 amazon.com
        ping -a -c 1 ns2
      SHELL
    end
  end