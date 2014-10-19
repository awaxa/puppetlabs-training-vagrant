# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'ipaddr'

STUDENTS = 1
MASTER = '192.168.168.10'
BRIDGE_INTERFACE = 'en0: Wi-Fi (AirPort)'

IPS = [ IPAddr.new(MASTER) ]
STUDENTS.times do
  IPS << IPS.last.succ
end

Vagrant.configure('2') do |config|
  config.vm.box = 'centos-6.5-pe-3.3.2-ptb2.12'
  config.vm.box_url = 'https://downloads.puppetlabs.com/training/centos-6.5-pe-3.3.2-ptb2.12-vagrant-vbox.box'

  config.vm.synced_folder '.', '/vagrant', disabled: true

  config.cache.disable! if Vagrant.has_plugin?('vagrant-cachier')

  config.vm.define :master do |master|
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
      student.vm.provider 'virtualbox' do |vb|
        vb.customize ['modifyvm', :id, '--memory', '512']
      end
      student.vm.network :public_network,
        bridge: BRIDGE_INTERFACE,
        ip: IPS[i]
    end
  end

end
