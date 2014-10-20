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
- `MASTER_VCPUS` and `MASTER_RAM` set resource allocation

## usage
    $ git clone https://github.com/awaxa/puppetlabs-training-vagrant.git
    Cloning into 'puppetlabs-training-vagrant'...
    $ cd puppetlabs-training-vagrant
    $ vagrant up
    Bringing machine 'master' up with 'virtualbox' provider...

## automated master setup
Set the `COURSE` environment variable to install PE and the `pltraining-classroom` module, add `include classroom::course::$COURSE` to the default node definition in `site.pp`, and do an agent run to start `ntpd`.  Requires internet access.

    $ COURSE=fundamentals vagrant up
    Bringing machine 'master' up with 'virtualbox' provider...
