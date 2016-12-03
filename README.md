# Docker Swarm Vagrant

This is a simple Vagrantfile which can be used to spin few nodes with Docker 1.12+ installed. You
can play with Docker Swarm on it. Boxes are Ubuntu Trusty amd64. 

# Customize

By default `vagrant up` spins three machines: manager, worker1 and worker2. You can adjust how many
machines you want in the `Vagrantfile`. The `instances` array contains hashes representing every
worker machine, keys are `:name` and `:ip`. Manager by default has address "192.168.10.2", workers have consecutive ips. 

```ruby
instances = [{ :name => "slave1", :ip => "192.168.10.3"}, 
             {:name => "slave2", :ip => "192.168.10.4" }]

```

`/etc/hosts` on every machine is populated with ip and name values so names are resolved on every 
machine. This mechanism is not idempotent, reprovisioning will append the hosts again. 

# Auto mode

By default, vagrant will create pure machines with docker installed. You can run 
`env=AUTO_START_SWARM=true vagrant up` to provision swarm automatically. You will get an already running Docker swarm cluster.

#  Play

After starting swarm, you can use my testing Docker image to play with. It is called `darek/goweb` and is a super simple Web app, displaying the hostname, and a version. There are three tags: `1.0`, `2.0` and `latest`. They can be used to play with swarm rolling update feature. The container exposes port 8080. 


# Author 
Inspired by `denverdino/docker-swarm-mode-vagrant` and `lowescott/learning-tools` repos. 

Dariusz Dwornikowski @tdi