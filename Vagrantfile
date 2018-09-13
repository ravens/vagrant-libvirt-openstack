# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box_check_update = false


   # this node will be responsible for deployment 
   config.vm.define :deployhost do |deployhost|

    deployhost.vm.box = "generic/ubuntu1604"
    deployhost.vm.provider :libvirt do |domain|
      domain.memory = 2048
      domain.cpus = 2
    end


    deployhost.vm.network :public_network,
      :dev => "eno1"

    deployhost.vm.provision "file", source: "openstack_user_config.yml", destination: "/tmp/openstack_user_config.yml"
    deployhost.vm.provision "file", source: "ssh_key", destination: "/tmp/ssh_key"
    deployhost.vm.provision "file", source: "ssh_key.pub", destination: "/tmp/ssh_key.pub"

    deployhost.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt-get -qy dist-upgrade
     apt-get -qy install aptitude build-essential git ntp ntpdate openssh-server python-dev sudo
     mkdir -p /root/.ssh
     cat /tmp/ssh_key.pub >> /root/.ssh/authorized_keys
     mv /tmp/ssh_key.pub /root/.ssh/id_rsa.pub
     mv /tmp/ssh_key /root/.ssh/id_rsa 
     chmod go-rwx /root/.ssh/id_rsa
     git clone -b 18.0.0.0rc2 https://github.com/openstack/openstack-ansible.git /opt/openstack-ansible
     cd /opt/openstack-ansible && scripts/bootstrap-ansible.sh
     cp -r /opt/openstack-ansible/etc/openstack_deploy /etc/
     cp /tmp/openstack_user_config.yml /etc/openstack_deploy/
     echo "openstack_service_publicuri_proto: http" >> /etc/openstack_deploy/user_variables.yml
     cd /opt/openstack-ansible && ./scripts/pw-token-gen.py --file /etc/openstack_deploy/user_secrets.yml
    SHELL



    # configure network VLAN10 as management interface on the management bridge br-mgmt
    deployhost.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook-network-deployhost.yml"
    end

    deployhost.vm.provision :reload

    deployhost.vm.provision "shell", inline: <<-SHELL
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-infrastructure.yml --syntax-check
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-hosts.yml
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-infrastructure.yml
       cd /opt/openstack-ansible/playbooks &&  openstack-ansible setup-openstack.yml
    SHELL

  end

  # reproducing https://docs.openstack.org/project-deploy-guide/openstack-ansible/newton/app-config-test.html
  # and following https://docs.openstack.org/project-deploy-guide/openstack-ansible/rocky/deploymenthost.html
  config.vm.define :infra1 do |infra1|

    infra1.vm.box = "generic/ubuntu1604"
    infra1.vm.provider :libvirt do |domain|
      domain.memory = 4096
      domain.cpus = 2
    end

    # physical interface
    #infra1.vm.network :private_network, :ip => "192.168.91.91"
    infra1.vm.network :public_network,
      :dev => "eno1"

    # openstack mirror, if needed
    # git clone -b 18.0.0.0rc2 https://git.openstack.org/openstack/openstack-ansible /opt/openstack-ansible
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
    SHELL

    
    # configure network VLAN10 as management interface on the management bridge br-mgmt
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

    #compute1.vm.network :private_network, :ip => "192.168.91.92"
    compute1.vm.network :public_network,
      :dev => "eno1"

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
       domain.memory = 2048
       domain.cpus = 1
       domain.nested = true
       domain.storage :file, :size => '100G'
       #domain.volume_cache = 'none'
    end

    #storage1.vm.network :private_network, :ip => "192.168.91.93"
    storage1.vm.network :public_network,
	       :dev => "eno1"

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
