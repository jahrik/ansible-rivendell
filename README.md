# jahrik.rivendell

[![CI/CD](https://github.com/jahrik/ansible-rivendell/actions/workflows/cicd.yml/badge.svg)](https://github.com/jahrik/ansible-rivendell/actions/workflows/cicd.yml)
[![Ansible Galaxy](https://img.shields.io/badge/ansible--galaxy-jahrik.rivendell-blue?logo=ansible)](https://galaxy.ansible.com/ui/standalone/roles/jahrik/rivendell/)

Builds [Rivendell](https://www.rivendellaudio.org/) radio automation software from source, on top
of an XFCE desktop, for a dedicated broadcast-automation box.

**Target platform: EL7 (CentOS 7 / RHEL 7).** CentOS 7 reached end-of-life in June 2024 -- there is
no newer EL release Rivendell 2.15.1 has been validated to build against here. See
[Proposed follow-ups](#proposed-follow-ups) below; this role intentionally keeps targeting EL7 for
now rather than guessing at an EL9 port.

## Usage

Include the role in your playbook:

```yaml
- hosts: all
  become: true
  roles:
    - jahrik.rivendell
```

Or run the bundled thin wrapper directly:

```bash
ansible-playbook -i inventory.ini playbook.yml
```

Galaxy role dependencies (EPEL, NTP, MySQL -- see [Dependencies](#dependencies)) run automatically
before this role's own tasks.

Each area can be re-applied on its own with tags: `user`, `hosts`, `sudo`, `dev_tools`, `repos`,
`xfce`, `install`, `misc` -- e.g. `--tags rivendell:xfce`.

## Dependencies

Declared in `requirements.yml` / `meta/main.yml`, installed automatically as role dependencies:

| Role | Replaces |
|---|---|
| `geerlingguy.repo-epel` | vendored `epel` role |
| `geerlingguy.ntp` | vendored `ntp` role |
| `geerlingguy.mysql` | vendored `mysql` role |

The NUX (Dextop) repo has no Galaxy equivalent and is handled directly in `tasks/repos.yml`.

## Variables

Key variables (see `defaults/main.yml` for the full list):

| Variable | Default | Description |
|---|---|---|
| `user.name` | `rd-user` | Linux account that runs Rivendell and logs in to XFCE. |
| `user.password` | `CHANGEME` | Override via Ansible Vault -- never commit a real password. |
| `rivendell_sudo_enabled` | `false` | Opt-in passwordless sudo for `user.name`. |
| `rivendell.version` | `2.15.1` | Rivendell source version to build. |
| `rivendell.pkgs` | see defaults | Build dependencies. |
| `desktop.pkgs` | `['@X Window system', '@Xfce']` | XFCE package groups (yum). |
| `nux_repo_url` | li.nux.ro Dextop RPM | NUX repo package URL. |
| `ntp_servers` | pool.ntp.org | Passed to `geerlingguy.ntp`. |
| `mysql_root_password`, `mysql_users` | `CHANGEME` | Passed to `geerlingguy.mysql` -- vault these. |

Set real values (credentials, hostnames) in your own `group_vars`/`host_vars`, and put every
`CHANGEME` in Ansible Vault -- never commit a real password.

## Testing

```bash
uv sync
uv run ansible-galaxy install -r requirements.yml
uv run yamllint .
uv run ansible-lint
uv run ansible-playbook playbook.yml --syntax-check
```

CI runs lint and a syntax check only -- see [AGENTS.md](AGENTS.md) for why.

## Proposed follow-ups

- Migrate the target platform to Rocky/Alma Linux 9 once Rivendell 4.x (which supports newer EL)
  has been validated against this role. This is a deliberate follow-up, not done here.
- Add a Molecule scenario once there's an EL image (or Rivendell version) that can actually
  converge in a container -- see [AGENTS.md](AGENTS.md) for why there isn't one today.

## License

GPL-3.0-or-later

## Author

jahrik@gmail.com
