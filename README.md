PuppetDB and Puppet Master Spike
================================

Prerequisites:
--------------

* Virtualbox 4.2.x+ - [See downloads](https://www.virtualbox.org/wiki/Downloads)
* Vagrant 1.3.x+ - [See downloads](http://downloads.vagrantup.com/)
* Centos 6.3 64-bit Vagrant box (or similar RHEL/Centos version) - [Download from here](https://dl.dropbox.com/u/7225008/Vagrant/CentOS-6.3-x86_64-minimal.box)
    * Note: `vagrant up` will download the box for you the first time if you don't have it.

First: `vagrant up puppetdb` to build the VM instance.
Then: `vagrant ssh puppetdb` to access the VM instance.

The `provision/puppetdb.sh` script will be run automatically to provision the machine.

On your host:
-------------

* Open a browser to `http://localhost:8080`
* You should see the PuppetDB dashboard with 1 node

API test:
---------

    # On the VM, type (remote HTTPS requires that you specify a certificate, a private key, and a CA certificate).
    curl -H "Accept: application/json" http://localhost:8080/facts/`hostname -f`

Sample API queries:
-------------------

    # On the VM
    cd /vagrant/queries
    
    # Run a query
    ./NODE_list_all.sh
    
    # See the query details
    less NODE_list_all.sh

Further steps:
--------------

* Create some puppet agents and register them with the master so that the puppet DB can collect some useful node data.
* Get HTTPS API working for remote API requests
* Play around with the API endpoints

Puppet Modules used:
--------------------

* `puppet module install theforeman/foreman -i puppet/common-modules`
* `puppet module install theforeman/puppet -i puppet/common-modules`
* `puppet module install puppetlabs/puppetdb -i puppet/common-modules`


Puppetboard
===========

Another VM can be created to try out [Puppetboard](https://github.com/nedap/puppetboard)

First: `vagrant up puppetboard` to build the VM instance.
Note: The `provision/puppetboard.sh` script will be run automatically to provision the machine.

Then: `vagrant ssh puppetboard` to access the VM instance.

On the VM:
----------

    cd puppetboard

    # Start the web app in the background
    nohup python dev.py &

On your host:
-------------

* Open a browser to `http://localhost:5000`


Dashboard
==========

This is an example dashboard using [Dashing](http://shopify.github.io/dashing/), a dashboard framework.
The dashboard shows how to integrate with PuppetDB for building custom apps.

First: `vagrant up dashboard` to build the VM instance.
Note: The `provision/dashboard.sh` script will be run automatically to provision the machine.

Then: `vagrant ssh dashboard` to access the VM instance.

On the VM:
----------

    sudo su
    source ~/.bash_profile
    cd /vagrant/dashboard/webapp
    nohup bundle exec dashing start &

On your host:
-------------

* Open a browser to `http://localhost:3030`


Exported Resources
==================

PuppetDB can be used for exporting resources from nodes so that other nodes can use this information in their manifests.

For example, monitoring hosts with Nagios, see [Exported Resources with Nagios](http://docs.puppetlabs.com/guides/exported_resources.html).

Machines `dashboard.example.com` and `puppetboard.example.com` export nagios host and service entries to PuppetDB.
The `puppetmaster.example.com` machine then collects these exported resources, dynamically creating nagios configuration files so
that the nagios can monitor these hosts.

See the Puppet module `puppet/modules/nagios` and the node manifests for further details.

TODO
====
* Mount /vagrant/puppet/modules and /vagrant/puppet/manifest dir to /etc/puppet/manifests and /etc/puppet/modules
* -or- create script which rsyncs the files to /etc/puppet (useful for making breaking changes for demo, etc)
* Provision puppetdb machine with puppet apply provisioner
* DRY up the provisioning scripts (or replace with puppet agent provisioner)
* Create a DNS instance to avoid /etc/hosts and again, to have another puppetised node in PuppetDB.
* Mount /etc/cache/yum to a local dir to avoid downloading RPMs over and over when rebuilding VMs (see: [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier))
