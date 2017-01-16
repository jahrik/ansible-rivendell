# Rivendell installation with ansible

Table of Contents
=================

   * [Rivendell installation with ansible](#rivendell-installation-with-ansible)
      * [Requirements](#requirements)
      * [Rivendell](#rivendell)
      * [Ansible](#ansible)
      * [Update the inventory.ini file](#update-the-inventoryini-file)
      * [Run ansible](#run-ansible)
      * [Roles](#roles)
         * [User](#user)
         * [sudo : Allow rd-user to have passwordless sudo](#sudo--allow-rd-user-to-have-passwordless-sudo)
         * [hosts](#hosts)
            * [Set hostname](#set-hostname)
            * [Update /etc/hosts](#update-etchosts)
         * [ntp](#ntp)
         * [epel : Install EPEL repo](#epel--install-epel-repo)
         * [nux : Install NUX repo](#nux--install-nux-repo)
         * [dev_tools : Install development tools](#dev_tools--install-development-tools)
         * [xfce](#xfce)
            * [Install desktop](#install-desktop)
            * [Enable GUI on startup](#enable-gui-on-startup)
            * [Generate ~/.xinitrc file](#generate-xinitrc-file)
            * [Enable autologin for rd-user](#enable-autologin-for-rd-user)
            * [Enable desktop manager service](#enable-desktop-manager-service)
         * [rivendell](#rivendell-1)
            * [Install rivendell dependencies](#install-rivendell-dependencies)
            * [Download rivendell-{{ rivendell.version }} from source](#download-rivendell--rivendellversion--from-source)
            * [Installation](#installation)
            * [Generate /etc/rd.conf](#generate-etcrdconf)
            * [Add "rd-user" to audio groups](#add-rd-user-to-audio-groups)
         * [MySQL](#mysql)
            * [Master](#master)
            * [Slave](#slave)
            * [Replication](#replication)
         * [Misc. tasks](#misc-tasks)
            * [Set timezone](#set-timezone)
            * [Add "en_US.UTF-8" to locale.conf](#add-en_usutf-8-to-localeconf)
            * [Stop and disable firewalld](#stop-and-disable-firewalld)
      * [Vagrant lab](#vagrant-lab)
      * [Errors](#errors)
         * [Unknown/unsupported storage engine: InnoDB](#unknownunsupported-storage-engine-innodb)
         * [Failed to open the relay log './mariadb-relay-bin.000001'](#failed-to-open-the-relay-log-mariadb-relay-bin000001)

## Requirements

CentOS 7.2

## Rivendell

* [http://www.rivendellaudio.org/](http://www.rivendellaudio.org/)
* [http://rivendell.tryphon.org/wiki/Main_Page](http://rivendell.tryphon.org/wiki/Main_Page)


## Ansible

* [http://docs.ansible.com/ansible/intro_installation.html](http://docs.ansible.com/ansible/intro_installation.html)

## Update the inventory.ini file

ex:
```
[master]
rivendell-01 ansible_host='192.168.1.133' ansible_user=root

[slave]
rivendell-02 ansible_host='192.168.1.134' ansible_user=root

[rd:children]
master
slave
```

## Run ansible

```
ansible-playbook site.yml --ask-pass
```

## Roles

### User

Set the default rd-user in group_vars/all.yml

This user will be used for:
* Default gdm login desktop user
* Sudo access
* Running rivendell
* Will be added to all groups used by rivendell (audio, etc)

group_vars/all.yml 
```
---
user:
  name: rd-user
  uid: 1000
  group: rd-user
  shell: /usr/bin/bash
  groups: [rivendell, wheel, audio, jackuser]
  password: changeme
```

After making changes to user variables you can run this role again to update the machines.
```
ansible-playbook site.yml --ask-pass --tags user
ansible-playbook -l master site.yml --ask-pass --tags user
ansible-playbook -l slave site.yml --ask-pass --tags user
```

### sudo : Allow rd-user to have passwordless sudo
Currently set to have passwordless sudo

Disabled at this time.

### hosts

#### Set hostname

Configurable in the inventory.ini file

Will also "echo 'hostname' > /etc/hostname" for each machine

#### Update /etc/hosts
All hosts that are defined with "ansible_host" in the inventory.ini file

will be dynamically added to every machines /etc/hosts file.

ex. /etc/hosts created for defined vagrant machines above.
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.33.133 rivendell-01
192.168.33.134 rivendell-02
```

### ntp
Install, configure, and enable ntp.

ntp servers can be configured in:

group_vars/all.yml
```
ntp_server: [0.pool.ntp.org, 1.pool.ntp.org]
```

### epel : Install EPEL repo
Required for:
  - rpmfusion
  - soundtouch{-devel}
  - et al.

### nux : Install NUX repo
Required for:
  - libmad
  - libmad-devel
  - twolame
  - twolame-devel 
  - lame
  - lame-devel

### dev_tools : Install development tools
Required for:
  - c++
  - etc
  - tools for installing rivendell from source

### xfce

#### Install desktop
[xfce](https://www.xfce.org/) is a lightweight desktop environment for UNIX-like operating systems.

It aims to be fast and low on system resources, while still being visually appealing and user friendly.

This role is essentially running the following command.
```
yum groups install X Window system
yum groups install Xfce
```

Configured in vars/RedHat.yml
```
---
desktop:
  pkgs:
  - '@X Window system'
  - '@Xfce'
  session: xfce4-session
  dm: gdm.service

```

#### Enable GUI on startup
A symlink is created:
 - from "/usr/lib/systemd/system/graphical.target"
 - to "/etc/systemd/system/default.target"

Enabling graphical login on a CentOS box.

#### Generate ~/.xinitrc file
An .xinitrc file is placed in the default rd-user's home directory containing the startup-session

This variable is also configurable at desktop.dm in vars/RedHat.yml

#### Enable autologin for rd-user
A custom.conf file is templated to /etc/gdm/custom.conf allowing automatic login.

This can be edited at templates/custom.conf.j2
```
[daemon]
AutomaticLogin=rd-user
AutomaticLoginEnable=True
...
...
```

#### Enable desktop manager service
gdm.service is enabled at startup

### rivendell

#### Install rivendell dependencies

Rivendell requirements can be configured at vars/RedHat.yml
```
rivendell:
  pkgs:
  - libogg-devel
  - vorbis-tools
  - libvorbis-devel
  - flac-devel
  - libmad
  - libmad-devel
  - twolame
  - twolame-devel 
  - lame
  - lame-devel
  - alsa-lib-devel
  - jack-audio-connection-kit
  - jack-audio-connection-kit-devel
  - qt
  - qt-devel
  - qt-mysql
  - qt3
  - qt3-devel
  - qt3-MySQL
  - libsamplerate
  - libsamplerate-devel
  - libsndfile
  - libsndfile-devel
  - cdparanoia
  - libXmu-devel
  - cdparanoia-devel
  - cdparanoia-libs
  - id3lib
  - id3lib-devel
  - libcurl
  - libcurl-devel  
  - pam-devel
  - soundtouch
  - soundtouch-devel

```

####  Download rivendell-{{ rivendell.version }} from source
Version can be specified at vars/RedHat.yml
```
rivendell:
  version: 2.15.1
```

#### Installation
All this will be handled.
  - Unarchive
  - Configure
  - Make package
  - daemon reload

Create rivendell directories
  - /var/run/rivendell
  - /home/rd-user/rdlogs

#### Generate /etc/rd.conf
Configure Rivendell

```

```

#### Add "rd-user" to audio groups
More groups can be added in:

group_vars/all.yml
```
user:
  groups:
    - audio
    - jackuser
```

### MySQL

#### Master

#### Slave

#### Replication

### Misc. tasks

#### Set timezone
configure in group_vars/all.yml
```
time_zone: US/Mountain
```

#### Add "en_US.UTF-8" to locale.conf
configure in group_vars/all.yml
```
locale: en_US.UTF-8
```

#### Stop and disable firewalld
For now firewalld is simply being disabled.

Might come back to this and add functionality to allow mysql ports, etc...

## Vagrant lab

```
vagrant up rivendell-01
ansible-playbook -l vagrant site.yml
```

## Errors

### Unknown/unsupported storage engine: InnoDB
```
rm /var/lib/mysql/ib_logfile*
rm: remove regular file ‘/var/lib/mysql/ib_logfile0’? y
rm: remove regular file ‘/var/lib/mysql/ib_logfile1’? y

systemctl start mariadb.service
```

### Failed to open the relay log './mariadb-relay-bin.000001'
Reset slave server
```
mysql >
RESET SLAVE;

```
