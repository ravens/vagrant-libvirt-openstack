# Vagrant-based virtual test openstack deployment 

## Todos

- Add another VM to act as a router for the br-mgmt VLAN. 
Use a 4th small VM as deployment machine. It looks like it does not work so far due to /etc/hosts

## dependencies

We are using an Ubuntu LTS 18.04 based system with Vagrant (version of vagrantup.com) and libvirt capable user.

We need QEMU/libvirt with KVM support:
```
sudo apt-get install qemu libvirt-bin qemu-kvm ebtables dnsmasq
sudo adduser $USER libvirt # you need to reload your session 
```

We need ansible for provisioning and 2 plugins for configuring the network interfaces of our target VMs:
```
sudo apt-get install ansible
ansible-galaxy install mrlesmithjr.ansible-openvswitch
ansible-galaxy install mrlesmithjr.config-interfaces
```

And some dependencies for building vagrant plugins written in ruby :
```
sudo apt-get install ruby-dev libvirt-dev
```

We need vagrant and some plugins
```
wget https://releases.hashicorp.com/vagrant/2.1.5/vagrant_2.1.5_x86_64.deb # or more recent
sudo dpkg -i vagrant_2.1.5_x86_64.deb
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-reload
```

We need to generate a SSH key for the cluster:
```
ssh-keygen -f ssh_key -P ""
```

You need then to adjust the Vagrantfile to reflect your physical ethernet adapter you wanna use to connect. It is currently configured to 'ens18f1'.

To change it to "enp0s25":
```
sed -i s/"ens18f1"/"enp0s25"/g Vagrantfile
```

You probably want to adjust the memory and CPU requirements to match your machine capabilities, as well as the public IP which is defined at 192.168.50.69.

NOTE: At this moment you need to have a router with the IP 172.29.236.1 on the VLAN 10 for the openstack ansible script to complete his job.

## openstack deployment

We are using Openstack Rock test deployment configuration here :
https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/deploymenthost.html

To start the entire process : 
```
vagrant up
```

After a couple of hours, Openstack admin interface will be listening on IP : https://192.168.50.69/

## how this actually works

We use vagrant to generate 3 VMs : 
- an infrastructure node that will basically contains the entire openstack components, from where the deployment is happening
- a compute node 
- a storage node with a 100G disk

The network is mapped automatically to the physical interface 'ens18f1' of the system I am using using macvlans.

The VM are generic ubuntu 18.04. We then use some [ansible recipes](https://github.com/mrlesmithjr/ansible-config-interfaces) to configure the network bridges that the openstack-ansible is expected to see. 


