# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'ipaddr'

BOX = 'https://downloads.puppetlabs.com/training/centos-6.5-pe-3.3.2-ptb2.12-vagrant-vbox.box'
BRIDGE_INTERFACE = 'en0: Wi-Fi (AirPort)'
MASTER = '192.168.168.10'
MASTER_RAM = 2048
MASTER_VCPUS = 2
STUDENTS = 1

BOX_NAME = BOX.split('/').last.sub('.box','')

IPS = [ IPAddr.new(MASTER) ]
STUDENTS.times do
  IPS << IPS.last.succ
end

MASTER_SCRIPT = <<MASTER_SCRIPT
#!/bin/bash

export MASTER_HOSTNAME='master.puppetlabs.vm'

# Setup the hostname for the system
export RUBYLIB=/usr/src/puppet/lib:/usr/src/facter/lib
/usr/src/puppet/bin/puppet resource host "$MASTER_HOSTNAME" \
  ensure=present \
  host_aliases="${MASTER_HOSTNAME/.*/}" \
  ip=#{MASTER}
unset RUBYLIB

/bin/sed -i "s/HOSTNAME.*/HOSTNAME=$MASTER_HOSTNAME/" /etc/sysconfig/network
/bin/hostname "$MASTER_HOSTNAME"

passwd -l root

sed -i s/enabled=1/enabled=0/ /etc/yum.repos.d/{CentOS-Base,epel}.repo
MASTER_SCRIPT

SETUP_CLASSROOM_MASTER = <<SETUP_CLASSROOM_MASTER
#!/bin/bash

export MASTER_HOSTNAME='master.puppetlabs.vm'

export q_all_in_one_install=y
export q_database_host=localhost
export q_database_install=y
export q_database_port=5432
export q_database_root_password=T3fXWQlfNDPiZA0zewA8
export q_database_root_user=pe-postgres
export q_install=y
export q_pe_database=y
export q_puppet_cloud_install=n
export q_puppet_enterpriseconsole_auth_database_name=console_auth
export q_puppet_enterpriseconsole_auth_database_password=AXWnz9TL0UPVB2PyOLRv
export q_puppet_enterpriseconsole_auth_database_user=console_auth
export q_puppet_enterpriseconsole_auth_password=puppetlabs
export q_puppet_enterpriseconsole_auth_user_email=admin@puppetlabs.com
export q_puppet_enterpriseconsole_database_name=console
export q_puppet_enterpriseconsole_database_password=6A98ZMWaooJa78Eo6q5T
export q_puppet_enterpriseconsole_database_user=console
export q_puppet_enterpriseconsole_httpd_port=443
export q_puppet_enterpriseconsole_install=y
export q_puppet_enterpriseconsole_master_hostname="$MASTER_HOSTNAME"
export q_puppet_enterpriseconsole_smtp_host=localhost
export q_puppet_enterpriseconsole_smtp_password=
export q_puppet_enterpriseconsole_smtp_port=25
export q_puppet_enterpriseconsole_smtp_use_tls=n
export q_puppet_enterpriseconsole_smtp_user_auth=n
export q_puppet_enterpriseconsole_smtp_username=
export q_puppet_symlinks_install=y
export q_puppetagent_certname="$MASTER_HOSTNAME"
export q_puppetagent_install=y
export q_puppetagent_server="$MASTER_HOSTNAME"
export q_puppetdb_database_name=pe-puppetdb
export q_puppetdb_database_password=3i1YuvUqaY48ruwKmeVz
export q_puppetdb_database_user=pe-puppetdb
export q_puppetdb_hostname=$MASTER_HOSTNAME
export q_puppetdb_install=y
export q_puppetdb_port=8081
export q_puppetmaster_certname="$MASTER_HOSTNAME"
export q_puppetmaster_dnsaltnames="${MASTER_HOSTNAME/.*/},${MASTER_HOSTNAME},puppet,puppet.puppetlabs.vm"
export q_puppetmaster_enterpriseconsole_hostname=localhost
export q_puppetmaster_enterpriseconsole_port=443
export q_puppetmaster_install=y
export q_run_updtvpkg=n
export q_vendor_packages_install=y

# Run the master installation with an empty file given the exports above
/root/puppet-enterprise/puppet-enterprise-installer -a /etc/motd

puppet module install pltraining-classroom

export MANIFEST=/etc/puppetlabs/puppet/manifests/site.pp
grep -q classroom::course $MANIFEST || sed -i '/node default {/a\  include classroom::course::#{ENV['COURSE']}' $MANIFEST

puppet agent --test
SETUP_CLASSROOM_MASTER

UBUNTU_MANIFEST = <<UBUNTU_MANIFEST
resources { 'host': purge => true }
host { 'master.puppetlabs.vm':
  ip           => '#{MASTER}',
  host_aliases => [ 'puppet' ],
}
host { 'ubuntu.puppetlabs.vm':
  ip           => $::ipaddress_lo,
  host_aliases => [ 'localhost' ],
}
exec { '/bin/hostname ubuntu.puppetlabs.vm': }
exec { '/bin/sed -i s/localhost/ubuntu.puppetlabs.vm/ /etc/puppetlabs/puppet/puppet.conf': }
service { 'pe-puppet':
  ensure => stopped,
  enable => false,
}
UBUNTU_MANIFEST

Vagrant.configure('2') do |config|
  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.cache.disable! if Vagrant.has_plugin?('vagrant-cachier')

  config.vm.define :master do |master|
    master.vm.box = BOX_NAME
    master.vm.box_url = BOX
    master.vm.provider 'virtualbox' do |vb|
      vb.customize ['modifyvm', :id, '--memory', MASTER_RAM]
      vb.customize ['modifyvm', :id, '--cpus', MASTER_VCPUS]
      vb.customize ['modifyvm', :id, '--ioapic', 'on']
    end
    master.vm.network :public_network,
      bridge: BRIDGE_INTERFACE,
      ip: IPS[0]
    master.vm.provision 'shell', inline: MASTER_SCRIPT
    master.vm.provision 'shell', inline: SETUP_CLASSROOM_MASTER if ENV['COURSE']
  end

  (1..STUDENTS).each do |i|
    config.vm.define "student#{i}".to_sym do |student|
      student.vm.box = BOX_NAME
      student.vm.box_url = BOX
      student.vm.provider 'virtualbox' do |vb|
        vb.customize ['modifyvm', :id, '--memory', '512']
      end
      student.vm.network :public_network,
        bridge: BRIDGE_INTERFACE,
        ip: IPS[i]
    end
  end

  config.vm.define :ubuntu, autostart: false do |ubuntu|
    ubuntu.vm.box = 'puppetlabs/ubuntu-12.04-64-puppet-enterprise'
    ubuntu.vm.network :public_network,
      bridge: BRIDGE_INTERFACE
    ubuntu.vm.provision 'shell', inline: "puppet apply -e \"#{UBUNTU_MANIFEST}\""
  end
end
