# Rivendell installation with ansible

Table of Contents
=================

   * [Rivendell installation with ansible](#rivendell-installation-with-ansible)
      * [Requirements](#requirements)
      * [Rivendell](#rivendell)
      * [Ansible](#ansible)
      * [Update the inventory.ini file](#update-the-inventoryini-file)
      * [Run ansible](#run-ansible)
      * [What's happening](#whats-happening)
         * [Enter user password [changeme]:](#enter-user-password-changeme)
         * [User](#user)
         * [sudo : Allow rd-user to have passwordless sudo](#sudo--allow-rd-user-to-have-passwordless-sudo)
         * [epel : Install EPEL repo](#epel--install-epel-repo)
         * [nux : Install NUX repo](#nux--install-nux-repo)
         * [dev_tools : Install development tools](#dev_tools--install-development-tools)
         * [xfce : Install desktop](#xfce--install-desktop)
         * [xfce : Enable GUI on startup](#xfce--enable-gui-on-startup)
         * [xfce : Generate ~/.xinitrc file](#xfce--generate-xinitrc-file)
         * [xfce : Enable autologin for rd-user](#xfce--enable-autologin-for-rd-user)
         * [xfce : Enable desktop manager service](#xfce--enable-desktop-manager-service)
         * [rivendell : Install rivendell dependencies](#rivendell--install-rivendell-dependencies)
         * [rivendell : Download rivendell-{{ rivendell.version }} from source](#rivendell--download-rivendell--rivendellversion--from-source)
      * [Vagrant lab](#vagrant-lab)

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
[vagrant]
rivendell-01.dev ansible_user=root
```

## Run ansible

```
ansible-playbook site.yml 
```

## What's happening

### Enter user password [changeme]: 

Set the password for the default rd-user

### User

Set the default rd-user in vars/all.yml

This user will be used for:
* Default gdm login desktop user
* Sudo access
* Rivendell source package installation
* Running rivendell
* Will be added to all groups used by rivendell (audio, etc)

vars/all.yml 
```
---
user:
  name: rd-user
  uid: 1001
  group: users
  shell: /usr/bin/bash
```

### sudo : Allow rd-user to have passwordless sudo
Currently set to have passwordless sudo

### epel : Install EPEL repo
Required for rpmfusion, soundtouch{-devel}, et al.

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

### xfce : Install desktop
[xfce](https://www.xfce.org/) is a lightweight desktop environment for UNIX-like operating systems. It aims to be fast and low on system resources, while still being visually appealing and user friendly.

This role is essentially running the following command.
```
yum groups install X Window system Xfce
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

### xfce : Enable GUI on startup
A symlink is created:
 - from "/usr/lib/systemd/system/graphical.target"
 - to "/etc/systemd/system/default.target"

Enabling graphical login on a CentOS box.

### xfce : Generate ~/.xinitrc file
An .xinitrc file is placed in the default rd-user's home directory containing the startup-session

This variable is also configurable at desktop.dm in vars/RedHat.yml

### xfce : Enable autologin for rd-user
A custom.conf file is templated to /etc/gdm/custom.conf allowing automatic login.

This can be edited at templates/custom.conf.j2
```
[daemon]
AutomaticLogin=rd-user
AutomaticLoginEnable=True
...
...
```

### xfce : Enable desktop manager service
gdm.service is enabled at startup

### rivendell : Install rivendell dependencies

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

### rivendell : Download rivendell-{{ rivendell.version }} from source
Version can be specified at vars/RedHat.yml
```
rivendell:
  version: 2.15.1
```

## Vagrant lab

```
vagrant up rivendell-01
ansible-playbook -l vagrant site.yml
```
