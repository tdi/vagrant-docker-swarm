# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

auto = ENV['AUTO_START_SWARM'] || false
# Increase numworkers if you want more than 3 nodes
numworkers = 2

# VirtualBox settings
# Increase vmmemory if you want more than 512mb memory in the vm's
vmmemory = 512
# Increase numcpu if you want more cpu's per vm
numcpu = 1

instances = []

(1..numworkers).each do |n| 
  instances.push({:name => "worker#{n}", :ip => "192.168.10.#{n+2}"})
end

manager_ip = "192.168.10.2"

File.open("./hosts", 'w') { |file| 
  instances.each do |i|
    file.write("#{i[:ip]} #{i[:name]} #{i[:name]}\n")
  end
}

http_proxy = ""
# Proxy configuration
if ENV['http_proxy']
	http_proxy = ENV['http_proxy']
	https_proxy = ENV['https_proxy']
end

no_proxy = "localhost,127.0.0.1,#{manager_ip}"
instances.each do |instance|
    no_proxy += ",#{instance[:ip]}"
end

# Vagrant version requirement
Vagrant.require_version ">= 1.8.4"

# Check if the necessary plugins are installed
if not http_proxy.to_s.strip.empty?
	required_plugins = %w( vagrant-proxyconf )
	plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
	if not plugins_to_install.empty?
    	puts "Installing plugins: #{plugins_to_install.join(' ')}"
    	if system "vagrant plugin install #{plugins_to_install.join(' ')}"
        	exec "vagrant #{ARGV.join(' ')}"
    	else
        	abort "Installation of one or more plugins has failed. Aborting."
    	end
	end
end

Vagrant.configure("2") do |config|
    config.vm.provider "virtualbox" do |v|
     	v.memory = vmmemory
  	v.cpus = numcpu
    end
    
    config.vm.define "manager" do |i|
      i.vm.box = "ubuntu/trusty64"
      i.vm.hostname = "manager"
      i.vm.network "private_network", ip: "#{manager_ip}"
      # Proxy
      if not http_proxy.to_s.strip.empty?
        i.proxy.http     = http_proxy
        i.proxy.https    = https_proxy
        i.proxy.no_proxy = no_proxy
      end
      i.vm.provision "shell", path: "./provision.sh"
      if File.file?("./hosts") 
        i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
        i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
      end 
      if auto 
        i.vm.provision "shell", inline: "docker swarm init --advertise-addr #{manager_ip}"
        i.vm.provision "shell", inline: "docker swarm join-token -q worker > /vagrant/token"
      end
    end 

  instances.each do |instance| 
    config.vm.define instance[:name] do |i|
      i.vm.box = "ubuntu/trusty64"
      i.vm.hostname = instance[:name]
      i.vm.network "private_network", ip: "#{instance[:ip]}"
      # Proxy
      if not http_proxy.to_s.strip.empty?
        i.proxy.http     = http_proxy
        i.proxy.https    = https_proxy
        i.proxy.no_proxy = no_proxy
      end
      i.vm.provision "shell", path: "./provision.sh"
      if File.file?("./hosts") 
        i.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
        i.vm.provision "shell", inline: "cat /tmp/hosts >> /etc/hosts", privileged: true
      end 
      if auto
        i.vm.provision "shell", inline: "docker swarm join --advertise-addr #{instance[:ip]} --listen-addr #{instance[:ip]}:2377 --token `cat /vagrant/token` #{manager_ip}:2377"
      end
    end 
  end
end
