# ansible-rivendell

Ansible role to install and configure [Rivendell](http://www.rivendellaudio.org/), an open-source radio broadcast automation system.

Targets CentOS 7 and handles everything from system dependencies and XFCE desktop configuration to MySQL replication and Rivendell source compilation.

## Quick Start

1. Define your host inventory in `inventory.ini` under `[master]` and `[slave]`.
2. Review variables in `group_vars/all.yml` and `vars/RedHat.yml`.
3. Run the playbook:

```bash
ansible-playbook site.yml --ask-pass
```

## Testing

Testing is driven by Molecule. Run tests via the `uv` environment:

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```

## Overview of Components

- **User**: Creates default `rd-user` (with autologin) and adds to `rivendell`, `audio`, and `jackuser` groups.
- **Desktop**: Installs XFCE and ensures graphical target runs on startup.
- **Rivendell**: Installs dependencies (Jack, ALSA, QT, etc.), downloads source, compiles, and generates `/etc/rd.conf`.
- **MySQL**: Sets up master/slave database replication for Rivendell.
- **NTP & Network**: Syncs time via NTP and dynamically updates `/etc/hosts` for all inventory nodes.
