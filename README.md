# Docker Swarm Vagrant

This is a simple Vagrantfile which can be used to spin few nodes with Docker 1.12+ installed. You
can play with Docker Swarm on it. Boxes are Ubuntu Trusty amd64. 

*Note*: This fork updates the original in providing two additional files, Vagrantefile.XENIAL and provision.sh.XENIAL. Replace the originals with these files if you wish to run a newer version of Docker in the Xenial version of Ubuntu. There is an issue with Docker Swarm when using the original files. See this:

[https://github.com/moby/moby/issues/34165]

So far as I have seen, using the Xenial versions of the files to create and provision the Vagrant nodes fixes this problem.

# Docker Swarm

Docker Swarm is a Docker clustering solution, it turns multiple physical (or virtual) hosts into a one cluster, which practically behaves as a single Docker host. Swarm additionally gives you tools and mechiasms to easily scale your containers and create managed services with automatic load balancing to the exposed ports. 

Swarm uses [Raft Consensus Algortihm](http://thesecretlivesofdata.com/raft/) to manage the cluster state. Swarm can tolerate `(N-1)/2` failures and needs `(N/2)+1` nodes to agree on values. 

# Customize

By default `vagrant up` spins up 3 machines: `manager`, `worker1`, `worker2`. You can adjust how many
workers you want in the `Vagrantfile`, by setting the `numworkers` variable. Manager, by default, has address "192.168.10.2", workers have consecutive ips. 

```ruby
numworkers = 2
```

If your provisioner is `Virtualbox`, you can modify the vm allocations for memory and cpu by changing these variables:

```ruby
vmmemory = 512
```

```ruby
numcpu = 1
```


`/etc/hosts` on every machine is populated with an IP address and a name of every other machine, so that names are resolved within the cluster. This mechanism is not idempotent, reprovisioning will append the hosts again. 

# Auto mode

By default, vagrant will create pure machines with docker installed. You can run 
`AUTO_START_SWARM=true vagrant up` to provision swarm automatically. You will get an already running Docker swarm cluster.

# Play

After starting swarm, you can use my testing Docker image to play with. It is called `darek/goweb` and is a super simple Web app, displaying the hostname, and a version. There are three tags: `1.0`, `2.0` and `latest`. They can be used to play with swarm rolling update feature. The container exposes port 8080. 

Go to the master node and start docker swarm:

```bash
   (host)# vagrant ssh manager
(manager)# docker swarm init --advertise-addr 192.168.10.2

docker swarm join \
--token SWMTKN-1-59h28hcbb8gzs2xs24oyh7hjvc7fp8skjzvnpw9cksmp96m4y2-35er9ai3u1f1ae5esb7x8l1hx \
192.168.10.2:2377
```

Now join the swarm on the nodes with the command from the manager, do it on both nodes. You can verify that nodes are in the cluster by doing `docker nodes ls` on the manager.

```bash
vagrant@manager:~$ docker node ls
ID                           HOSTNAME  STATUS  AVAILABILITY  MANAGER STATUS
0rtuyz07e0wazmxvoed1llmx3 *  manager   Ready   Active        Leader
1iab1w9m5znyzmh7hpzyn7rrw    worker2   Ready   Active
4mmca5rxrxxhb0tb7s5du5hpe    worker1   Ready   Active
```

Now create a web service, with 1 replica, on any of the nodes:

```bash
docker service create --name web --replicas 1 --publish 8080:8080 darek/goweb:1.0
```
You can see the status od the service with `docker service ps web`. 
```
vagrant@manager:~$ docker service  ps web
ID                         NAME   IMAGE            NODE     DESIRED STATE  CURRENT STATE           ERROR
7m0s3ikyjyvps4xgpk655uv9k  web.1  darek/goweb:1.0  manager  Running        Running 16 seconds ago
```

You can scale up the service if you want:

```bash 
docker service scale web=4
```

Now, let us verify it works, docker swarm will load balance requests to all of the service instances:

```bash
vagrant ssh worker1
sudo apt-get install httpie -y
http localhost:8080
```
Do the http command serveral times, you will notice different hostnames every time. This is because swarm handler load balancing for us. 

Let's do a rolling update of the service, first let's scale it to 10 - you already know how. 
First we will update the defintion of the service to say that updates will have a delay of 5s.  

```bash
docker service update --update-delay 5s web
```
Let's us update the service now:

```bash
docker service update --image darek/goweb:2.0 web
```
Now docker will update service one by one with a 5s delay between the updates. If you want to introduce parallelism to updates you can specify if with the `--update-parallelism` flag. 

```bash 
docker service update --update-parallelism 2 web
```

Let's see how it goes:

```bash
vagrant@manager:~$ docker service ps web
ID                         NAME        IMAGE            NODE     DESIRED STATE  CURRENT STATE                    ERROR
7m0s3ikyjyvps4xgpk655uv9k  web.1       darek/goweb:1.0  manager  Running        Running 8 minutes ago
f49k99apn8yzve0uh8zsyigw6  web.2       darek/goweb:1.0  worker1  Running        Running 7 minutes ago
8t9x9qqqxztzd8msuccbx43mg  web.3       darek/goweb:2.0  manager  Running        Running 11 seconds ago
38pl9czfhaztifjsb605wi2m9   \_ web.3   darek/goweb:1.0  worker1  Shutdown       Shutdown 16 seconds ago
5y4iaxmv1jhdw5b7qroessmr9  web.4       darek/goweb:1.0  worker2  Running        Running 7 minutes ago
2zhfna5m15cd482hbt06kc139  web.5       darek/goweb:2.0  worker1  Running        Preparing 1 seconds ago
3ydym0s1iq0psko60zi66ojvs   \_ web.5   darek/goweb:1.0  manager  Shutdown       Shutdown less than a second ago
9feao0z26oc75qgoktuc16pro  web.6       darek/goweb:1.0  manager  Running        Running about a minute ago
14tsult13blurmrh2klf7sm7i  web.7       darek/goweb:2.0  worker2  Running        Running 43 seconds ago
1qtcsaohat94puoym9i6zsadf   \_ web.7   darek/goweb:1.0  manager  Shutdown       Shutdown 48 seconds ago
8d2qnv19urq2yzlmqzud41ffa  web.8       darek/goweb:1.0  worker2  Running        Running about a minute ago
9shytfqyotuuyaj5hiddnymd4  web.9       darek/goweb:1.0  worker2  Running        Running about a minute ago
9kr6ehtv093cnmdxo55sy4pl9  web.10      darek/goweb:2.0  worker1  Running        Running 27 seconds ago
1yvwgb25d8fbarrabyeqha843   \_ web.10  darek/goweb:1.0  worker1  Shutdown       Shutdown 31 seconds ago
```

Some services are already updated. Nice.

You can also inspect the service by `docker service inspect --pretty web`. 

```bash
vagrant@manager:~$ docker service inspect --pretty web
ID:		0iqjzgky9p7ilky60xff4ovgt
Name:		web
Mode:		Replicated
 Replicas:	10
Update status:
 State:		completed
 Started:	3 minutes ago
 Completed:	48 seconds ago
 Message:	update completed
Placement:
UpdateConfig:
 Parallelism:	1
 Delay:		10s
 On failure:	pause
ContainerSpec:
 Image:		darek/goweb:2.0
Resources:
Ports:
 Protocol = tcp
 TargetPort = 8080
 PublishedPort = 8080
```

After the update, you can verify that the version is really 2.0 with the httpie command. 
Let's now delete the service. 

```bash
docker service rm web
```

And shutdown the shop for good: `vagrant destroy --force`. 

# License 

MIT

# Author 
Inspired by `denverdino/docker-swarm-mode-vagrant` and `lowescott/learning-tools` repos. 

Dariusz Dwornikowski @tdi
