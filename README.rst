================
OpenStack Swift
================

Swift is a highly available, distributed, eventually consistent object/blob store. Organizations can use Swift to store lots of data efficiently, safely, and cheaply.

Sample pillars
==============

Swift proxy server
------------------

.. code-block:: yaml

    swift:
      common:
        cache:
          engine: memcached
          members:
          - host: 127.0.0.1
            port: 11211
          - host: 127.0.0.1
            port: 11211
        enabled: true
        version: kilo
        swift_hash_path_suffix: hash
        swift_hash_path_prefix: hash
      proxy:
        version: kilo
        enabled: true
        bind:
          address: 0.0.0.0
          port: 8080
        identity:
          engine: keystone
          host: 127.0.0.1
          port: 35357
          user: swift
          password: pwd
          tenant: service

Swift storage server
--------------------

.. code-block:: yaml

    swift:
      common:
        cache:
          engine: memcached
          members:
          - host: 127.0.0.1
            port: 11211
          - host: 127.0.0.1
            port: 11211
        version: kilo
        enabled: true
        swift_hash_path_suffix: hash
        swift_hash_path_prefix: hash
      object:
        enabled: true
        version: kilo
        bind:
          address: 0.0.0.0
          port: 6000
      container:
        enabled: true
        version: kilo
        allow_versions: true
        bind:
          address: 0.0.0.0
          port: 6001
      account:
        enabled: true
        version: kilo
        bind:
          address: 0.0.0.0
          port: 6002


To enable object versioning feature

.. code-block:: yaml

    swift:
      ....
      container:
        ....
        allow_versions: true
        ....

Ring builder
------------

.. code-block:: yaml

    parameters:
      swift:
        ring_builder:
          enabled: true
          rings:
            - name: default
              partition_power: 9
              replicas: 3
              hours: 1
              region: 1
              devices:
                - address: ${_param:storage_node01_address}
                  device: vdb
                - address: ${_param:storage_node02_address}
                  device: vdc
                - address: ${_param:storage_node03_address}
                  device: vdd
            - partition_power: 9
              replicas: 2
              hours: 1
              region: 1
              devices:
                - address: ${_param:storage_node01_address}
                  device: vdb
                - address: ${_param:storage_node02_address}
                  device: vdc

Documentation and Bugs
============================

To learn how to deploy OpenStack Salt, consult the documentation available
online at:

    https://wiki.openstack.org/wiki/OpenStackSalt

In the unfortunate event that bugs are discovered, they should be reported to
the appropriate bug tracker. If you obtained the software from a 3rd party
operating system vendor, it is often wise to use their own bug tracker for
reporting problems. In all other cases use the master OpenStack bug tracker,
available at:

    http://bugs.launchpad.net/openstack-salt

Developers wishing to work on the OpenStack Salt project should always base
their work on the latest formulas code, available from the master GIT
repository at:

    https://git.openstack.org/cgit/openstack/salt-formula-swift

Developers should also join the discussion on the IRC list, at:

    https://wiki.openstack.org/wiki/Meetings/openstack-salt


Development and testing
=======================

Development and test workflow with `Test Kitchen <http://kitchen.ci>`_ and
`kitchen-salt <https://github.com/simonmcc/kitchen-salt>`_ provisioner plugin.

Test Kitchen is a test harness tool to execute your configured code on one or more platforms in isolation.
There is a ``.kitchen.yml`` in main directory that defines *platforms* to be tested and *suites* to execute on them.

Kitchen CI can spin instances locally or remote, based on used *driver*.
For local development ``.kitchen.yml`` defines a `vagrant <https://github.com/test-kitchen/kitchen-vagrant>`_ or
`docker  <https://github.com/test-kitchen/kitchen-docker>`_ driver.

To use backend drivers or implement your CI follow the section `INTEGRATION.rst#Continuous Integration`__.

A listing of scenarios to be executed:

.. code-block:: shell

  $ kitchen list

  Instance                    Driver   Provisioner  Verifier  Transport  Last Action

  proxy-cluster-ubuntu-1404    Docker  SaltSolo     Inspec    Ssh        <Not Created>
  proxy-cluster-ubuntu-1604    Docker  SaltSolo     Inspec    Ssh        <Not Created>
  proxy-cluster-centos-71      Docker  SaltSolo     Inspec    Ssh        <Not Created>
  proxy-single-ubuntu-1404     Docker  SaltSolo     Inspec    Ssh        <Not Created>
  proxy-single-ubuntu-1604     Docker  SaltSolo     Inspec    Ssh        <Not Created>
  proxy-single-centos-71       Docker  SaltSolo     Inspec    Ssh        <Not Created>
  storage-cluster-ubuntu-1404  Docker  SaltSolo     Inspec    Ssh        <Not Created>
  storage-cluster-ubuntu-1604  Docker  SaltSolo     Inspec    Ssh        <Not Created>
  storage-cluster-centos-71    Docker  SaltSolo     Inspec    Ssh        <Not Created>

The `Busser <https://github.com/test-kitchen/busser>`_ *Verifier* is used to setup and run tests
implementated in `<repo>/test/integration`. It installs the particular driver to tested instance
(`Serverspec <https://github.com/neillturner/kitchen-verifier-serverspec>`_,
`InSpec <https://github.com/chef/kitchen-inspec>`_, Shell, Bats, ...) prior the verification is executed.


Usage:

.. code-block:: shell

 # list instances and status
 kitchen list

 # manually execute integration tests
 kitchen [test || [create|converge|verify|exec|login|destroy|...]] [instance] -t tests/integration

 # use with provided Makefile (ie: within CI pipeline)
 make kitchen

