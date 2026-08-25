<!--
SPDX-FileCopyrightText: 2023 Slavi Pantaleev
SPDX-FileCopyrightText: 2025 spatterIight
SPDX-FileCopyrightText: 2025, 2026 Suguru Hirahara

SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Jellyfin Ansible role

This is an [Ansible](https://www.ansible.com/) role which installs [Jellyfin](https://jellyfin.org/) to run as a [Docker](https://www.docker.com/) container wrapped in a systemd service.

This role *implicitly* depends on:

- [`com.devture.ansible.role.playbook_help`](https://github.com/devture/com.devture.ansible.role.playbook_help)
- [`com.devture.ansible.role.systemd_docker_base`](https://github.com/devture/com.devture.ansible.role.systemd_docker_base)

Check [`defaults/main.yml`](defaults/main.yml) for the full list of supported options.

💡 For an Ansible playbook which integrates this role and makes it easier to use, see the [Mother-of-All-Self-Hosting Ansible playbook](https://github.com/mother-of-all-self-hosting/mash-playbook).

## Limitations

Most LinuxServer docker images support non-root user operation and read-only capabilities. The Jellyfin image is not one of these. By default this role is configured to:

1. Run as root user
2. Run the container as read/write (NOT read-only)

Additionally, like all LinuxServer docker images, full `cap-drop` is not supported, and several capabilities related to permissions are added to the container:

- SETUID
- SETGID
- CHOWN
- FOWNER
- DAC_OVERRIDE

### The first-run setup wizard

Jellyfin ships with no accounts. Until somebody completes its first-run wizard, the wizard is open: `POST /Startup/User` followed by `POST /Startup/Complete` creates the administrator, and neither call requires authentication. Whoever makes those calls first owns the server. This is not unusual for self-hosted software and is not something this role can close — Jellyfin offers no setting to bind the wizard to a local address or to require a token — but two details are worth knowing before pointing a public hostname at a freshly deployed instance:

- Jellyfin sends `Access-Control-Allow-Origin: *` on the wizard endpoints and answers the CORS preflight for them, so the calls do not have to come from someone who can reach the port directly. A web page open in any browser that can resolve and reach the instance can make them.
- While the wizard is outstanding, `/System/Info` — which is authenticated afterwards — answers anonymously, reporting the server's internal paths and architecture.

Both were confirmed against `lscr.io/linuxserver/jellyfin:10.11.11`, and both stop the moment the wizard is completed. Complete it immediately after the first deployment, and prefer doing so before the instance is reachable from the internet.

## Development

### pre-commit

You can optionally install a Git pre-commit hook (via [mise](https://mise.jdx.dev/) + [prek](https://prek.j178.dev/)) that runs formatting and linting checks before each commit. See [`.pre-commit-config.yaml`](./.pre-commit-config.yaml) for which hooks are to be executed.

To install the hook, run the [`just`](https://github.com/casey/just) command below:

```sh
just prek-install-git-pre-commit-hook
```

### Molecule

This role supports [Molecule](https://docs.ansible.com/projects/molecule/), an Ansible testing framework designed for developing and testing Ansible collections, playbooks, and roles.

Refer to [this page](./molecule/README.md) for details about how to utilize it.
