# Rivendell installation with ansible

## Table of Contents
   * [Rivendell installation with ansible](#rivendell-installation-with-ansible)
         * [Requirements](#requirements)
         * [Rivendell](#rivendell)
         * [Install ansible](#install-ansible)
         * [Update the inventory.ini file with the master and client IP addresses](#update-the-inventoryini-file-with-the-master-and-client-ip-addresses)
         * [Run ansible](#run-ansible)
         * [Tags](#tags)
         * [Vagrant lab](#vagrant-lab)


### Requirements

CentOS 7.2

### Rivendell

### Install ansible

Debian
```
sudo apt-get install ansible
```

Alternatively you can install ansible using pip.
```
sudo apt-get intsall python-pip
sudo pip install ansible
```


### Update the inventory.ini file with the master and client IP addresses

ex:
```
[vagrant]
rivendell-01.dev ansible_user=root
```

### Run ansible

```
ansible-playbook site.yml 
```

### Tags

### Vagrant lab

```
vagrant up rivendell-01
ansible-playbook -l vagrant site.yml
```
