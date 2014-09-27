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

Copy `user-data.sample` to `user-data`.

```
cp user-data.sample user-data
```

Generate a new `etcd` token [here](https://discovery.etcd.io/new) and
change `discovery:` option in `data-sample`.

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

Create a shared folder.

```
mkdir -p ~/.containers/apollo
```

Environment
-----------

Set the environment you are working on.

```
export APOLLO_ENVIRONMENT=local
export APOLLO_TAG=$APOLLO_ENVIRONMENT
export DOCKER_REGISTRY="${APOLLO_ENVIRONMENT}.registry.apollolab.com.br:5000"
```

It recommended to add it to `~/.bashrc`. It's used in other projects.

Common address
--------------

As root set more one ip address to `eth0`.

```
ifconfig eth0:0 172.16.16.16
```

ssh
---

Add the following to `~/.ssh/config`:

```
Host coreos
  User     core
  HostName 172.16.16.101
  IdentityFile ~/.ssh/id_rsa.pub
```

Add your key:

```
ssh-add ~/.ssh/id_rsa
ssh-add  ~/.vagrant.d/insecure_private_key
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
```

Starting registry
-----------------

You will need a local `docker-registry`. For more info
[see](https://github.com/wiliamsouza/apollo/blob/develop/registry/README.md)

Fleet
-----

Configuring `fleet`.

```
export FLEETCTL_TUNNEL=172.16.16.101
```

```
fleetctl list-machines
MACHINE         IP              METADATA
b3dac926...     172.16.16.101    -
```

Reloading user-data
-------------------

Inside a machine run:

```
sudo coreos-cloudinit --from-file /var/lib/coreos-vagrant/vagrantfile-user-data
```
