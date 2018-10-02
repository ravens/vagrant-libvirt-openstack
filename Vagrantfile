# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false


  $linux_server_provisioning = <<-SCRIPT
     apt-get update
     apt-get -qy dist-upgrade
     apt-get -qy install bridge-utils debootstrap ifenslave ifenslave-2.6 lsof lvm2 ntp ntpdate openssh-server sudo tcpdump vlan python
     apt-get -qy install linux-image-extra-$(uname -r)
     echo 'bonding' >> /etc/modules
     echo '8021q' >> /etc/modules
     mkdir -p /root/.ssh
     cat /tmp/ssh_key.pub >> /root/.ssh/authorized_keys
  SCRIPT


  config.vm.define :deploy do |deploy|

    deploy.vm.box = "generic/ubuntu1604"
    deploy.vm.provider :libvirt do |domain|
      domain.memory = 1024 
      domain.cpus = 1 
    end

    # physical interface
    deploy.vm.network :public_network,
      :dev => "enp0s25"

    deploy.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    deploy.vm.provision "shell", inline: $linux_server_provisioning

    deploy.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook-network-deployhost.yml"
    end

    deploy.vm.provision "file", source: "user_variables.yml", destination: "/tmp/user_variables.yml"
    deploy.vm.provision "file", source: "openstack_user_config.yml", destination: "/tmp/openstack_user_config.yml"
    deploy.vm.provision "file", source: "ssh_key", destination: "/tmp/ssh_key"

    deploy.vm.provision "shell", inline: <<-SHELL 
     mv /tmp/ssh_key /root/.ssh/id_rsa 
     chmod go-rwx /root/.ssh/id_rsa
     git clone -b stable/rocky https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible
     cd /opt/openstack-ansible && scripts/bootstrap-ansible.sh
     cp -r /opt/openstack-ansible/etc/openstack_deploy /etc/
     cp /tmp/openstack_user_config.yml /etc/openstack_deploy/
     cp /tmp/user_variables.yml /etc/openstack_deploy/
     cd /opt/openstack-ansible && ./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
    SHELL

    deploy.vm.provision :reload

    deploy.vm.provision "shell", inline: <<-SHELL
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-everything.yml
    SHELL

  end
  # END OF DEPLOY

  # reproducing https://docs.openstack.org/project-deploy-guide/openstack-ansible/newton/app-config-test.html
  # and following https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/deploymenthost.html
  config.vm.define :infra1 do |infra1|

    infra1.vm.box = "generic/ubuntu1604"
    infra1.vm.provider :libvirt do |domain|
      domain.memory = 8192 
      domain.cpus = 4 
    end

    # physical interface
    infra1.vm.network :public_network,
      :dev => "enp0s25"

    infra1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    infra1.vm.provision "shell", inline: $linux_server_provisioning

    infra1.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook-network-infra1.yml"
    end

    infra1.vm.provision :reload

  end
  # END OF INFRA1

  # BEGIN of COMPUTE1
  config.vm.define :compute1 do |compute1|

    compute1.vm.box = "generic/ubuntu1604"
     compute1.vm.provider :libvirt do |domain|
       domain.memory = 2048 
       domain.cpus = 2 
       domain.nested = true
       #domain.volume_cache = 'none'
    end

    compute1.vm.network :public_network,
      :dev => "enp0s25"

    compute1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    compute1.vm.provision "shell", inline: $linux_server_provisioning

#    compute1.vm.provision "shell", inline: <<-SHELL
#     apt install libvirt-bin
#     modprobe kvm_intel nested=1
#     echo "options kvm_intel nested=1" >> /etc/modprobe.d/kvm.conf
#    SHELL


    # configure network VLAN10 as management interface on the management bridge br-mgmt
    compute1.vm.provision "ansible" do |ansible|
      ansible.playbook = "playbook-network-compute1.yml"
    end

    compute1.vm.provision :reload



  end
  # END of COMPUTE1

  # BEGIN of storage1
  config.vm.define :storage1 do |storage1|

    storage1.vm.box = "generic/ubuntu1604"
     storage1.vm.provider :libvirt do |domain|
       domain.memory = 1024 
       domain.cpus = 1
       domain.storage :file, :size => '100G'
       #domain.volume_cache = 'none'
    end

    storage1.vm.network :public_network,
	       :dev => "enp0s25"

    storage1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    storage1.vm.provision "shell", inline: $linux_server_provisioning

    storage1.vm.provision "shell", inline: <<-SHELL
     pvcreate --metadatasize 2048 /dev/vda
     vgcreate cinder-volumes /dev/vda
    SHELL

    storage1.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook-network-storage1.yml"
    end

    storage1.vm.provision :reload


  end
  # END of STORAGE1

end
