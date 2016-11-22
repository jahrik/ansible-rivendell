# Rivendell installation with ansible

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
```

### Run ansible

```
ansible-playbook site.yml 
ansible-playbook -l master site.yml 
ansible-playbook -l clients site.yml 
```

## Tags

### Vagrant lab

```
vagrant up
ansible-playbook -l vagrant site.yml
```
