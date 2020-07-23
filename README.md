ansible-zookeeper
=================

[![Build Status](https://travis-ci.org/AnsibleShipyard/ansible-zookeeper.svg?branch=master)](https://travis-ci.org/AnsibleShipyard/ansible-zookeeper)

ZooKeeper playbook for Ansible

Installation
-----------

```bash
ansible-galaxy install daltinn.ansible-zookeeper
```

Requirements
------------

Ansible version at least 1.6

Role Variables
--------------

```yaml
---
zookeeper_version: 3.4.12
zookeeper_url: http://www.us.apache.org/dist/zookeeper/zookeeper-{{zookeeper_version}}/zookeeper-{{zookeeper_version}}.tar.gz

# Flag that selects if systemd or upstart will be used for the init service:
# Note: by default Ubuntu 15.04 and later use systemd (but support switch to upstart)
zookeeper_debian_systemd_enabled: "{{ ansible_distribution_version|version_compare(15.04, '>=') }}"
zookeeper_debian_apt_install: false
# (Optional:) add custom 'ppa' repositories depending on the distro version (only with debian_apt_install=true)
# Example: to use a community zookeeper v3.4.8 deb pkg for Ubuntu 14.04 (where latest official is v3.4.5)
zookeeper_debian_apt_repositories:
  - repository_url: "ppa:ufscar/zookeeper"
    distro_version: "14.04"

apt_cache_timeout: 3600
zookeeper_register_path_env: false

client_port: 2181
init_limit: 5
sync_limit: 2
tick_time: 2000
zookeeper_autopurge_purgeInterval: 0
zookeeper_autopurge_snapRetainCount: 10
zookeeper_cluster_ports: "2888:3888"
zookeeper_max_client_connections: 60

data_dir: /var/lib/zookeeper
log_dir: /var/log/zookeeper
zookeeper_dir: /opt/zookeeper-{{zookeeper_version}} # or /usr/share/zookeeper when zookeeper_debian_apt_install is true
zookeeper_conf_dir: {{zookeeper_dir}} # or /etc/zookeeper when zookeeper_debian_apt_install is true
zookeeper_tarball_dir: /opt/src

zookeeper_hosts_hostname: "{{inventory_hostname}}"
# List of dict (i.e. {zookeeper_hosts:[{host:,id:},{host:,id:},...]})
zookeeper_hosts:
  - host: "{{zookeeper_hosts_hostname}}" # the machine running
    id: 1

# Dict of ENV settings to be written into the (optional) conf/zookeeper-env.sh
zookeeper_env: {}

# Controls Zookeeper myid generation
zookeeper_force_myid: yes
```

Example Playbook
----------------

```yaml
- name: Installing ZooKeeper
  hosts: all
  sudo: yes
  roles:
    - role: AnsibleShipyard.ansible-zookeeper
```

Example Retrieving Tarball From S3
----------------------------------

```yaml
- name: Installing ZooKeeper
  hosts: all
  sudo: yes
  vars:
    zookeeper_archive_s3_bucket: my-s3-bucket
    zookeeper_archive_s3_object: my/s3/directory/zookeeper-{{zookeeper_version}}.tar.gz
  roles:
    - role: AnsibleShipyard.ansible-zookeeper
```

Cluster Example
----------------

```yaml
- name: Zookeeper cluster setup
  hosts: zookeepers
  sudo: yes
  roles:
    - role: AnsibleShipyard.ansible-zookeeper
      zookeeper_hosts: "{{groups['zookeepers']}}"
```

Assuming ```zookeepers``` is a [hosts group](http://docs.ansible.com/ansible/intro_inventory.html#group-variables) defined in inventory file.

```inventory
[zookeepers]
server[1:3]
```

Custom IP per host group

```
zookeeper_hosts: "
    {%- set ips = [] %}
    {%- for host in groups['zookeepers'] %}
    {{- ips.append(dict(id=loop.index, host=host, ip=hostvars[host]['ansible_default_ipv4'].address)) }}
    {%- endfor %}
    {{- ips -}}"
```

See this sample [playbook](https://github.com/AnsibleShipyard/ansible-galaxy-roles/blob/master/playbook.yml)
which shows how to use this playbook as well as others. It is part of [ansible-galaxy-roles](https://github.com/AnsibleShipyard/ansible-galaxy-roles) and
serves as a curation (and thus an example) of all our ansible playbooks.

Test Role
---------

1. Install Molecule
2. Enshure dependency installed ()

```bash
molecule test
```

License
-------

see LICENSE file

Author Information
------------------

@AnsibleShipyard/developers and others.
