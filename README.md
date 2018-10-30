# Prerequisites
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
in the same time search and modify these lines:
```
ip = "192.168.33.#{i+100}"
config.vm.network :"public_network", ip: ip, bridge:"enp0s31f6"  
```
to get 5 machine, modify this line
```
$num_instances = 5
```
