# AGENTS.md

This file provides guidance to AI agents when working with the `ansible-rivendell` role.

## Overview

This is an Ansible role to install and configure Rivendell, an open-source radio broadcast automation system. It handles user setup, desktop environment (XFCE), Rivendell installation from source, and MySQL replication for CentOS 7 environments.

## Testing

Testing is driven by Molecule. Ensure dependencies are installed via `uv`:

```bash
uv sync
source .venv/bin/activate
yamllint .
ansible-lint
molecule test
```

## Conventions

- **Idempotency**: Ensure all tasks can be safely run multiple times.
- **Linting**: Follow `ansible-lint` and `yamllint` rules. 
- **FQCN**: Use Fully Qualified Collection Names (e.g., `ansible.builtin.yum` instead of `yum`).
- **Variables**: Define default values in `defaults/main.yml` or `vars/RedHat.yml` where appropriate.
