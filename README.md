Apollo coreos
=============

This uses CoreOS, Etcd, Fleet and confd to provides distributed configuration
and service discovery to Apollo api and web application.

Dependencies
------------

This repo uses Vagrant and Virtualbox to create three CoreOS virtual machine.

* [Virtualbox](https://www.virtualbox.org/) 4.3.10 or greater.
* [Vagrant](http://www.vagrantup.com/) 1.6 or greater.

Configuring
-----------

Copy `user-data.sample` to `data-sample`.

```
cp user-data.sample data-sample
```

Generate a new `etcd` token [here](https://discovery.etcd.io/new) and
change `discovery:` option in `data-sample`.

Change `DOCKER_REGISTRY` in `user-data` to point to your
local running `docker-registry` usually it will be local ip address.

Add your ssh public key in `user-data` changing `path` and `content` as
the example.

```
- path: /home/core/.ssh/authorized_keys.d/<USERNAME-HERE>
  permissions: 0600
  owner: core
  content: |
    <SSH-RSA-KEY-HERE>
```
Copy `config.rb.sample` to `config.rb`

```
cp config.rb.sample config.rb
```

ssh
---

Add the following to `~/.ssh/config`:

```
Host coreos
  User     core
  HostName 172.17.8.101
  IdentityFile ~/.ssh/id_rsa.pub
```

Add your key:

```
ssh-add ~/.ssh/id_rsa
```

Check if it is correctly loaded:

```
ssh-add -l
```

Starting the cluster
--------------------

It will start three nodes.

```
vagrant up
```

To get in a node use.

```
vagrant ssh apollo-01
vagrant ssh apollo-02
vagrant ssh apollo-03
```

Starting registry
-----------------

You will need a local `docker-registry`. For more info
[see](https://github.com/wiliamsouza/apollo/blob/develop/registry/README.md)

fleet
-----

Configuring `fleet`.

```
export FLEETCTL_TUNNEL=172.16.8.101
```

```
fleetctl list-machines
MACHINE         IP              METADATA
b3dac926...     172.17.8.101    -
```

Loading a service:

```
cd api/systemd/
ln -s api.service api@8000.service
```

```
fleetctl load api@8000.service
Job api.service loaded on b3dac926.../172.17.8.101
```

```
fleetctl list-units
UNIT            STATE   LOAD    ACTIVE          SUB     DESC           MACHINE
api@8000.service     loaded  loaded  inactive        dead    apollo-api    b3dac926.../172.17.8.101
```

```
fleetctl start api@8000.service
Job api.service launched on b3dac926.../172.17.8.101
```

```
fleetctl list-units
UNIT            STATE           LOAD    ACTIVE          SUB             DESC            MACHINE
api@8000.service     launched        loaded  activating      start-pre       apollo-api     b3dac926.../172.17.8.101
```

Reloading user-data
-------------------

Inside a machine run:

```
sudo coreos-cloudinit --from-file /var/lib/coreos-vagrant/vagrantfile-user-data
```
