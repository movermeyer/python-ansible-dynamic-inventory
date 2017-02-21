# ansible-dynamic-inventory
Generate ansible dynamic inventory from static inventory.  
Optionally, Replace the host list of ansible static inventory with ServiceAddress registered in consul service.  
[![Build Status](https://travis-ci.org/Yoshiyuki-Nakahara/python-ansible-dynamic-inventory.svg?branch=master)](https://travis-ci.org/Yoshiyuki-Nakahara/python-ansible-dynamic-inventory)
[![PyPI version](https://badge.fury.io/py/ansible-dynamic-inventory.svg)](https://badge.fury.io/py/ansible-dynamic-inventory)
![Python Version](https://img.shields.io/badge/python-2.7-blue.svg)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

# Assumed Use Case\
  - Dynamic inventory conversion from static inventory  
  - In the service operated using [Consul](https://www.consul.io/), inventory is dynamically generated without rewriting static inventory when host information changes dynamically, such as automatic failover

# References
  [Ansible inventory](http://docs.ansible.com/ansible/intro_inventory.html)  
  [Ansible Dynamic Inventory](http://docs.ansible.com/ansible/intro_dynamic_inventory.html)  
  [Consul Catalog Service](https://www.consul.io/docs/agent/http/catalog.html#catalog_service)  

# Installation
    $ yum install gcc python-devel openssl-devel python-pip
    $ pip install --upgrade pip setuptools
    $ pip install ansible-dynamic-inventory

# Prerequisite of Replace with Consul Service
  If the group name written in the ansible static inventory and the service name registered in the consul service are the same, the host name is replaced.

# Configuration
    # vi /etc/ansible_dynamic_inventory.ini

    [ansible]
    # path to inventory file or directory
    static_inventory_path = /path/to/ansible_inventory

    [consul]
    #url = http://localhost:8500/v1
    url =

    # Forcibly replace even if consul response the number of hosts is 0, yes or no
    # If you choose no, if the response from consul does not have host information, use the ansible static inventory setting as is
    force_replace_zero_hosts = no

# Usage
    # Stand alone execution
    $ /usr/bin/ansible-dynamic-inventory

    # As Ansible Dynamic Inventory execution
    $ ansible-playbook --inventory /usr/bin/ansible-dynamic-inventory /path/to/playbook.yml

# Stand alone execution result example
    ex. ansible:static_inventory_path = ${this repository}/sample_inventory

    $ /usr/bin/ansible-dynamic-inventory | jq .
    {
      "all": {
        "hosts": [
          "10.10.10.12",
          "10.10.10.13",
          "10.10.10.14",
          "10.10.10.15",
          "10.10.10.11"
        ],
        "children": [
          "ungrouped",
          "_mysql_replication_config",
          "mysql_backup_storage",
        ],
        "vars": {
          "datacenter": "vagrant",
          "net_cidr": "10.10.10.0/24"
        }
      },
      "_mysql_replication_config": {
        "hosts": [
          "10.10.10.12",
          "10.10.10.13",
          "10.10.10.14",
          "10.10.10.15",
          "10.10.10.11"
        ],
        "children": [
          "mysql_replication_master",
          "mysql_replication_slave",
          "mysql_replication_backup",
          "mysql_failover"
        ],
        "vars": {
          "mysql_replication_user": "root",
          "mysql_master_host": "{{groups.mysql_replication_master[0]}}",
          "mysql_master_group_name": "mysql_replication_master",
          "mysql_version": "5.6.34",
          "mysql_replication_group_name": "single",
          "mysql_slave_group_name": "mysql_replication_slave"
        }
      },
      "mysql_backup_storage": {
        "hosts": [
          "10.10.10.15"
        ],
        "vars": {
          "mysql_backup_target_host": "{{groups.mysql_replication_backup[0]}}",
          "mysql_backup_storage_cron_file": "../../../../varfiles/vagrant/mysql/backup_storage/mysql_backup"
        }
      },
      "mysql_replication_slave": {
        "hosts": [
          "10.10.10.13",
          "10.10.10.14"
        ]
      },
      "mysql_replication_backup": {
        "hosts": [
          "10.10.10.15"
        ]
      },
      "ungrouped": {},
      "mysql_failover": {
        "hosts": [
          "10.10.10.11"
        ],
        "vars": {
          "mysql_failover_config_file": "../../../../varfiles/vagrant/mysql/failover/config.yml"
        }
      },
      "mysql_replication_master": {
        "hosts": [
          "10.10.10.12"
        ]
      }
    }

# License
MIT License
