# AGENTS.md

Guidance for AI agents working in `ansible-rivendell`.

## Overview

Ansible role (Galaxy: `jahrik.rivendell`) that builds
[Rivendell](https://www.rivendellaudio.org/) radio automation from source on top of an XFCE
desktop. `playbook.yml` at the repo root is a thin wrapper that `include_role`s this role by
directory, so `ansible-playbook playbook.yml` keeps working without the repo being checked out
under a `roles/` path.

This repo was converted from a 10-role playbook (`user, hosts, ntp, epel, nux, dev_tools, xfce,
rivendell, mysql, sudo`) into a single role. The generic pieces (EPEL, NTP, MySQL) are now Galaxy
role dependencies; the trivial ones (user, hosts, sudo, dev_tools) and the project-specific ones
(rivendell build, XFCE, NUX repo) are folded into `tasks/`.

**Target platform: EL7 (CentOS 7 / RHEL 7) only.** CentOS 7 is EOL (June 2024). This role
deliberately keeps targeting EL7 for now -- see README.md "Proposed follow-ups" for the Rocky/Alma
9 migration that should happen next, once Rivendell's build has been validated on a newer EL.

## Task Flow

`tasks/main.yml` dispatches each area via `include_tasks` with the `apply:` tag form, so a single
area can be re-applied with `--tags rivendell:<area>`:

| File | Tag | Notes |
|---|---|---|
| `tasks/user.yml` | `rivendell:user` | Creates the `rd-user` account and groups. |
| `tasks/hosts.yml` | `rivendell:hosts` | Sets hostname, updates `/etc/hosts`. |
| `tasks/sudo.yml` | `rivendell:sudo` | Opt-in only -- gated by `rivendell_sudo_enabled` (default `false`), matching upstream. |
| `tasks/dev_tools.yml` | `rivendell:dev_tools` | `@Development Tools` yum group. |
| `tasks/repos.yml` | `rivendell:repos` | NUX (Dextop) repo -- EPEL is a role dependency instead. |
| `tasks/xfce.yml` | `rivendell:xfce` | XFCE desktop, GDM autologin. |
| `tasks/install.yml` | `rivendell:install` | Download/build/install Rivendell from source. |
| `tasks/misc.yml` | `rivendell:misc` | Timezone, locale, disables firewalld. |

Galaxy role dependencies (`geerlingguy.repo-epel`, `geerlingguy.ntp`, `geerlingguy.mysql` -- see
`meta/main.yml`) run automatically before the tasks above.

## Conventions

- **Idempotency:** build/install tasks in `tasks/install.yml` gate on
  `not rd_admin.stat.exists` (checked once via `ansible.builtin.stat`) rather than a "changed"
  registered result, so they're safe to re-run and don't trip `no-handler` lint.
- **Handlers:** `systemctl daemon-reload` is a handler (`handlers/main.yml`), not an inline
  `command` task.
- **Dependencies:** use `uv` for Python package management; do not use `pip` directly.
- **OS constraint:** this role targets EL7 only. `ansible.builtin.yum` (not `dnf`) is used
  deliberately for `@Group` installs -- EL7 has no `dnf`, and modern `ansible-core` no longer ships
  a `yum` module under a canonical FQCN, so those tasks carry a `# noqa: fqcn[action-core]` with an
  explanatory comment. This is a direct symptom of running an EOL platform against current tooling
  -- see the Rocky/Alma 9 follow-up.
- **Facts:** use `ansible_facts['<fact>']`, never the bare `ansible_<fact>` magic vars;
  `ansible.cfg` sets `inject_facts_as_vars = False`.
- **FQCN:** all modules fully qualified (`ansible.builtin.*`, `community.general.*`).
- **Secrets:** `user.password`, `mysql_root_password`, and `mysql_users[].password` default to
  `CHANGEME` in `defaults/main.yml` -- override via Ansible Vault in a real `group_vars`/
  `host_vars` file, never commit a real value.
- **General rules:** abide by the global `AGENTS.md` guidelines (no hardcoded secrets or IPs).

## Testing

```bash
uv sync
uv run ansible-galaxy install -r requirements.yml
uv run yamllint .
uv run ansible-lint
uv run ansible-playbook playbook.yml --syntax-check
```

## CI/CD

- **lint** -- `yamllint` + `ansible-lint` (profile `production`).
- **syntax** -- `ansible-playbook playbook.yml --syntax-check`. No Molecule/converge job: this role
  installs a desktop environment and builds broadcast-audio software (JACK, ALSA) from source
  against a live init system -- nothing here safely converges in a container. Lint + syntax-check
  is the CI gate for this repo.
- **release** -- `needs: [lint, syntax]`, publishes to Ansible Galaxy on merge to `main`.
