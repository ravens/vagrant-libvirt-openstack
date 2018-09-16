# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false

  # reproducing https://docs.openstack.org/project-deploy-guide/openstack-ansible/newton/app-config-test.html
  # and following https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/deploymenthost.html
  config.vm.define :infra1 do |infra1|

    infra1.vm.box = "generic/ubuntu1604"
    infra1.vm.provider :libvirt do |domain|
      domain.memory = 16384 
      domain.cpus = 8 
    end

    # physical interface
    infra1.vm.network :public_network,
      :dev => "ens18f1"

    # openstack mirror, if needed
    # git clone -b 18.0.0.0rc2 https://git.openstack.org/openstack/openstack-ansible /opt/openstack-ansible
    infra1.vm.provision "file", source: "user_variables.yml", destination: "/tmp/user_variables.yml"
    infra1.vm.provision "file", source: "openstack_user_config.yml", destination: "/tmp/openstack_user_config.yml"
    infra1.vm.provision "file", source: "ssh_key", destination: "/tmp/ssh_key"
    infra1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    infra1.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt-get -qy dist-upgrade
     apt-get -qy install bridge-utils debootstrap ifenslave ifenslave-2.6 lsof lvm2 ntp ntpdate openssh-server sudo tcpdump vlan python
     apt-get -qy install linux-image-extra-$(uname -r)
     echo 'bonding' >> /etc/modules
     echo '8021q' >> /etc/modules
     mkdir -p /root/.ssh
     cat /tmp/ssh_key.pub >> /root/.ssh/authorized_keys
     mv /tmp/ssh_key /root/.ssh/id_rsa 
     chmod go-rwx /root/.ssh/id_rsa
     git clone -b 18.0.0.0rc2 https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible
     cd /opt/openstack-ansible && scripts/bootstrap-ansible.sh
     cp -r /opt/openstack-ansible/etc/openstack_deploy /etc/
     cp /tmp/openstack_user_config.yml /etc/openstack_deploy/
     cp /tmp/user_variables.yml /etc/openstack_deploy/
     cd /opt/openstack-ansible && ./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
    SHELL

    
    # configure network VLAN10 as management interface on the management bridge br-mgmt
    infra1.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook-network-infra1.yml"
    end

    infra1.vm.provision :reload

    infra1.vm.provision "shell", inline: <<-SHELL
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-infrastructure.yml --syntax-check
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-hosts.yml
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-infrastructure.yml
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-openstack.yml
    SHELL

  end
  # END OF INFRA1

  # BEGIN of COMPUTE1
  config.vm.define :compute1 do |compute1|

    compute1.vm.box = "generic/ubuntu1604"
     compute1.vm.provider :libvirt do |domain|
       domain.memory = 65536 
       domain.cpus = 28 
       domain.nested = true
       #domain.volume_cache = 'none'
    end

    compute1.vm.network :public_network,
      :dev => "ens18f1"

    compute1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    compute1.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get -qy dist-upgrade
    apt-get -qy install bridge-utils debootstrap ifenslave ifenslave-2.6 lsof lvm2 ntp ntpdate openssh-server sudo tcpdump vlan python
    apt-get -qy install linux-image-extra-$(uname -r)
    echo 'bonding' >> /etc/modules
    echo '8021q' >> /etc/modules
    mkdir -p /root/.ssh
    cat /tmp/ssh_key.pub >> /root/.ssh/authorized_keys
    SHELL

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
       domain.memory = 8192 
       domain.cpus = 4 
       domain.nested = true
       domain.storage :file, :size => '100G'
       #domain.volume_cache = 'none'
    end

    storage1.vm.network :public_network,
	       :dev => "ens18f1"

    storage1.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    storage1.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get -qy dist-upgrade
    apt-get -qy install bridge-utils debootstrap ifenslave ifenslave-2.6 lsof lvm2 ntp ntpdate openssh-server sudo tcpdump vlan python
    apt-get -qy install linux-image-extra-$(uname -r)
    echo 'bonding' >> /etc/modules
    echo '8021q' >> /etc/modules
    mkdir -p /root/.ssh
    cat /tmp/ssh_key.pub >> /root/.ssh/authorized_keys
    pvcreate --metadatasize 2048 /dev/vda
    vgcreate cinder-volumes /dev/vda
    SHELL

    # configure network VLAN10 as management interface on the management bridge br-mgmt
    storage1.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook-network-storage1.yml"
    end

    storage1.vm.provision :reload


  end
  # END of STORAGE1

end
