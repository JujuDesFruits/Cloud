made not in china by
Cédric Flamant & Julien Trijean
# Prerequisites
we started to do this, to get vagrant:
```
wget -c https://releases.hashicorp.com/vagrant/2.0.3/vagrant_2.0.3_x86_64.deb
sudo dpkg -i vagrant_2.0.3_x86_64.deb
git clone https://github.com/coreos/coreos-vagrant/
cd coreos-vagrant
vagrant plugin install vagrant-disksize
```
then to set 10GB to a disk for a box vagrant, add this line in Vagrantfile under the line **Vagrant.configure("2") do |config|**   :
```
config.disksize.size = '10GB'
```
At the same time search and modify these lines:
```
ip = "192.168.33.#{i+100}"
config.vm.network :"public_network", ip: ip, bridge:"enp0s31f6"  
```
to get 5 machines, modify this line
```
$num_instances = 5
```

# Mise en place
acces in one machine via ssh
```
vagrant ssh core-01 # "core-01" is an example
docker swarm init --advertise-addr 192.168.56.101 #IP of th VM
```
then copy the given line to add worker in other machine and to add manager use this line to copy the manager token:
```
docker swarm join-token -q manager
```
BE CAREFUL, if you shut down your vm or our computer, It may append a lot of errors like this:
```
core@core-01 ~ $ docker node ls    
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
6g4d3gbp1dbixdu4yvvg9ke3v *   core-01             Ready               Active              Leader              18.06.1-ce
wu0ibw53ioer7p4fz84gx7pjh     core-02             Unknown             Active              Unreachable         18.06.1-ce
j9a3vdlws4z64sfs98xr1lwxn     core-03             Ready               Active              Reachable           18.06.1-ce
5u44tw20kxz7e76uczuqxgho3     core-04             Down                Active                                  18.06.1-ce
9biu72yn4g7v5ric5ezc1cclf     core-05             Down                Active                                  18.06.1-ce
```
In that case I recommend you to type the command line just under and restart all your manipulation:
```
vagrant destroy
```
# Dumb Service - Part 1
you only have to get back the repository of an older project:
[link](https://github.com/It4lik/B3-Cloud-2018/tree/master/tp1/app)

## Weave Cloud
after creating an account and following [steps](https://www.weave.works/docs/cloud/latest/install/docker-swarm/) to use it, you can see the structur of your docker swarm.

**Answer**
* weave cloud works using a temporary container, services start up a container with a docker client and docker socket connected to the swarm. It also works because there is no privileged in Swarm.

## Ceph
We will only follow the line recommended in the original [documentation](https://github.com/It4lik/B3-Cloud-2018/tree/master/tp1)

**Answer**
* distributed filesystem provides services that map the directories and file names of the file system, to acces them on any machine that are connected by this services.
* I think without trying it, that we can automate the deployement using Ansible and vagrantfile. I've found this [link](https://github.com/ceph/ceph-ansible) and this should work.
*

## NFS
find the line:
```
 #config.vm.synced_folder
```
on Vagrantfile and delete the **#**.
Also add this line to enable host-only network
```
config.vm.network "private_network", ip: ip, type: "dhcp"
```
**Answer**
* Network File System is a file sharing protocol, it doesn't define anything about the underlying file system.
* our method is already an automate one

### Registry - Part 1
**Answer**
* The service is lunch into the host where you run the command line usually or into another node with more cpu available, but you can specify where to start it.
to know where the services is running  just type:
```
docker stack services <NAME>
```
* It's the work of the orchestrator in the swarm manager to connect each node with there services.
![https://docs.docker.com/engine/swarm/images/service-lifecycle.png](https://docs.docker.com/engine/swarm/images/service-lifecycle.png)

# Dumb Service - Part 2

## Keepalived
**Answer**
* *--net=host* is used to define an outside gate to interact with the inside docker swarm network. *nsenter* is a tool used to acces into namespace, It is used to debug a service.
to use it you need this two line:
```
# to know PID of the container
PID=$(docker inspect --format {{.State.Pid}} <name>)
# to enter the container
nsenter --target $PID --mount --uts --ipc --net --pid
```
* *KEEPALIVED_PRIORITY* or Virtual Router Redundancy Protocol give a device's priority for becoming the master default device. To becoming more priority, just decrease the value associated.

## Show me your metrics
**Answer**
* *collector* in this context means that tools expose and group informations from each container/services
* we gonna list each tool with a short description of their job :
  - prometheus, It collects metrics
  from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is  observed to be true
  - grafana is a website showing metrics collected by prometheus, it's organised by diagram and values
  - node-exporter is Prometheus exporter for hardware and OS metrics exposed by UNIX kernels, with pluggable metric collectors.
  - cadvisor, It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics.
  - alertmanager handles alerts sent by client applications. It takes care of deduplicating, grouping, and routing them to the correct receiver integration. It also takes care of silencing and inhibition of alerts.
  - unsee is the altert dashboard for Prometheus alertmanager.
  - caddy, It scans Docker metadata looking for labels indicating that the service or container should be exposed on caddy.

## Traefik
**Answer**
* Traefik as a reverse proxy, create a connection between micro services and the outside network. Here is an illustration from the official web site:
![https://docs.traefik.io/img/architecture.png](https://docs.traefik.io/img/architecture.png)
