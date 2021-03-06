# Ansible MMDD - Mesos, Marathon, and Docker

A work-in-progress for setting up a quick Vagrant||OpenStack and Ansible-based Apache Mesos, Marathon, and Docker installation.

The masters run:

* Zookeeper
* Mesos-master
* Marathon

The slaves run:

* Mesos-slave
* Docker

##Requirements

###If using OpenStack

* [shooz](https://github.com/curtisgithub/shooz) - The shooz Ansible inventory script is required if using OpenStack instead of Vagrant to provision the virtual machines. Just link it in for now to the root of the playbook.
* [mesos_facts](https://github.com/curtisgithub/ansible-mesos_facts-module) - Should be in ./library/mesos_facts. Just link it in for now.


##Notes

* Each Mesos master must have a host_var entry with a unique zoo_id variable.

```bash
curtis$ cat host_vars/mm3
zoo_id: 3
```

* Vagrant - Hardcoded IPs based on what is in the Vagrantfile
* Vagrant - Assumes an apt-cacher-ng proxy

##Todo

* *git subtree* for shooz and mesos_facts
* OpenStack - configure gw as an apt-cache?
* OpenStack - configure gw
* OpenStack - security groups: allow 5050, 8080, 2181 tcp
* DONE: Deploy to OpenStack private cloud
* Setup ability to use right eth interface whether openstack or vagrant (logic in templates?)
* DONE: mesos (zookeeper) + marathon
* DONE: enable cgroups in mesos
* Marathon authentication? (And marathon authenticate to mesos?) - "--http_credentials user:pass"
* docker
* http server to grab zip files from to launch applications?
* /var/log/marathon probably isn't being used
* [java heap size for zookeeper?](http://zookeeper.apache.org/doc/r3.1.2/zookeeperAdmin.html)
* DONE: cgroups config:

```bash
W0820 21:19:03.126446  8711 mesos_containerizer.cpp:116] The 'cgroups' isolation flag is deprecated, please update your flags to '--isolation=cgroups/cpu,cgroups/mem'.
```

##OpenStack

### Security groups

Ensure security groups are open so that the servers can talk to one another on ports: 5050, 8080, 2181, and 5051. I'm sure you could tighten that down a little in a production environment. I've at least limited those ports to the OpenStack private network.

Also you will likely need to open the marathon default ports.

Example shown below, where the private network is 10.2.0.0/20.

```bash
curtis$ nova secgroup-list-rules default
+-------------+-----------+---------+-------------+--------------+
| IP Protocol | From Port | To Port | IP Range    | Source Group |
+-------------+-----------+---------+-------------+--------------+
| icmp        | -1        | -1      | 0.0.0.0/0   |              |
| tcp         | 22        | 22      | 0.0.0.0/0   |              |
| tcp         | 5050      | 5050    | 10.2.0.0/20 |              |
| tcp         | 8080      | 8080    | 10.2.0.0/20 |              |
| tcp         | 2181      | 2181    | 10.2.0.0/20 |              |
| tcp         | 5051      | 5051    | 10.2.0.0/20 |              |
| tcp         | 31000     | 32000   | 10.2.0.0/20 |              |
| tcp         | 2888      | 2888    | 10.2.0.0/20 |              |
| tcp         | 3888      | 3888    | 10.2.0.0/20 |              |
+-------------+-----------+---------+-------------+--------------+
```

###ssh proxy

Ensure netcat-tradititional is installed on the openstack-gw (assuming it's Ubuntu 14.04).

```bash
host openstack-gw
   Hostname some.public.ip
   User ubuntu
# Where 10.2.*.* is your tenant/project private ip space
host 10.2.*.*
   ProxyCommand ssh -q openstack-gw netcat %h 22
```

##Issues

* set quorum to 2 with three masters? "If quorum is 2, you should have 3 masters"
