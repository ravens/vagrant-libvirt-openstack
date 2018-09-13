# Vagrant-based virtual test openstack deployment 

## todo

Fix final phase of deployment which is still failing.
Add another VM to act as a router for the br-mgmt VLAN. 


## dependencies

We are using an Ubuntu LTS 18.04 based system with Vagrant (version of vagrantup.com) and libvirt capable user.

We need QEMU/libvirt :
```
sudo apt-get install qemu libvirt-bin ebtables dnsmasq qemu-kvm
sudo adduser $USER libvirt # you need to reload your session 
```

We need vagrant and some plugins
```
wget https://releases.hashicorp.com/vagrant/2.1.5/vagrant_2.1.5_x86_64.deb
sudo dpkg -i vagrant_2.1.5_x86_64.deb
vagrant plugin install vagrant-libvirt
vagrant plugin install vagrant-reload
```

We need to generate a SSH key for the cluster:
```
ssh-keygen -f ssh_key -P ""
```

You need then to adjust the Vagrantfile to reflect your physical ethernet adapter you wanna use to connect. It is currently configured to 'eno1'.

NOTE: At this moment you need to have a router with the IP 172.29.236.1 on the VLAN 10 for the openstack ansible script to complete his job.

## openstack deployment

We are using Openstack Rock test deployment configuration here :
https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/deploymenthost.html

We introduced the following changes :
- we have one virtual IP on the infra1 node in order to cope with some deployment bug at the deploy-infra.yml stage (i.e. if the VIP public/private are the same the deployment is simply not happening)

To start the entire process : 
```
vagrant up
```

## how this actually works

We use vagrant to generate 4 VM : 
- a deployment machine with ansible scripts from openstack-ansible project, currently configured for Rocky
- a infrastructure node that will basically contains the entire openstack components
- a compute node 
- a storage node with a 100G disk

The network is mapped automatically to the physical interface 'eno1' of the system I am using using macvlans.

The VM are generic ubuntu 18.04. We then use some [ansible recipes](https://github.com/mrlesmithjr/ansible-config-interfaces) to configure the network bridges that the openstack-ansible is expected to see. 


