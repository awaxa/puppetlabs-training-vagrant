# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'ipaddr'

STUDENTS = 1
MASTER = '192.168.168.10'
BRIDGE_INTERFACE = 'en0: Wi-Fi (AirPort)'
BOX = 'https://downloads.puppetlabs.com/training/centos-6.5-pe-3.3.2-ptb2.12-vagrant-vbox.box'

BOX_NAME = BOX.split('/').last.sub('.box','')

IPS = [ IPAddr.new(MASTER) ]
STUDENTS.times do
  IPS << IPS.last.succ
end

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
      vb.customize ['modifyvm', :id, '--memory', '2048']
      vb.customize ['modifyvm', :id, '--cpus', '2']
      vb.customize ['modifyvm', :id, '--ioapic', 'on']
    end
    master.vm.network :public_network,
      bridge: BRIDGE_INTERFACE,
      ip: IPS[0]
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
