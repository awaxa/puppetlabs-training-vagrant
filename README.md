# a vagrant environment for [Puppet Labs training courses](https://puppetlabs.com/services/training)

### note: this is designed for use by instructors, not students

## requirements
- [vagrant](https://www.vagrantup.com)
- [virtualbox](https://www.virtualbox.org)
  - vmware support will be added later

## configuration
Edit the variables at the top of the Vagrantfile
- `STUDENTS` number of student VMs
- `MASTER` IP address for master VM.  student VM `$n` will be assigned `MASTER + $n`
- `BRIDGE_INTERFACE` host interface for classroom network
- `BOX` training VM vagrant box URL

## usage
    $ git clone https://github.com/awaxa/puppetlabs-training-vagrant.git
    $ cd puppetlabs-training-vagrant
    $ vagrant up
