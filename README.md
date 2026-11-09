# Ansible Collection: `cube_m4st3r.homelab`

[![ci](https://github.com/cube-m4st3r/ansible-collection-homelab/actions/workflows/ci.yml/badge.svg)](https://github.com/cube-m4st3r/ansible-collection-homelab/actions/workflows/ci.yml)
[![pre-commit](https://img.shields.io/badge/pre--commit-enabled-brightgreen?logo=pre-commit)](https://pre-commit.com/)

Ansible roles that deploy self-hosted apps as **rootless Podman Quadlets**, which systemd then manages like any other
service. Built for [my homelab](https://github.com/cube-m4st3r/homelab), and generic enough for yours.

> **Status:** early development. Expect breaking changes until `1.0.0`.

## Roles

| Role | What it does | Status |
| --- | --- | --- |
| `podman_host` | Installs Podman, creates the service user, sets up subuid/subgid, lingering and the auto-update timer | planned |
| `quadlet_app` | Generic: takes a list of apps and writes `.container` / `.network` / `.volume` files, reloads systemd, removes apps that are no longer listed | planned |
| `cloudflared` | Cloudflare Tunnel connector | planned |
| `authentik` | Authentik server + worker + PostgreSQL | planned |
| `stalwart` | Stalwart mail server | planned |

The app roles build on `quadlet_app`. Each one knows its app's images and settings and hands them over as a ready-made list.

## Requirements

- ansible-core: <!-- TODO: set once meta/runtime.yml exists -->
- Managed hosts: systemd + Podman **4.4 or newer** (first version with Quadlet support)
- Collections: `containers.podman` (installed automatically as a dependency)

## Installation

From Galaxy (once released):

```bash
ansible-galaxy collection install cube_m4st3r.homelab
```

Or from Git, e.g. in a `requirements.yml`:

```yaml
collections:
  - name: https://github.com/cube-m4st3r/ansible-collection-homelab.git
    type: git
    version: main # pin a release tag once there is one
```

## Example

```yaml
- name: Deploy apps
  hosts: apps
  roles:
    - role: cube_m4st3r.homelab.podman_host
    - role: cube_m4st3r.homelab.quadlet_app
      vars:
        quadlet_apps:
          - name: whoami
            image: docker.io/traefik/whoami:latest
            ports: ["127.0.0.1:8080:80"]
            networks: [proxy]
```

Every role validates its input with an argument spec (`meta/argument_specs.yml`). Run
`ansible-doc -t role cube_m4st3r.homelab.<role>` to see all of its variables.

## Development

```bash
git clone https://github.com/cube-m4st3r/ansible-collection-homelab.git
cd ansible-collection-homelab
pre-commit install
```

- **Lint:** `pre-commit run --all-files` (ansible-lint, yamllint, gitleaks, actionlint)
- **Test:** Molecule scenarios under `extensions/molecule/`. Each one checks that a role converges and that a second run changes nothing.
- **Commits:** [Conventional Commits](https://www.conventionalcommits.org/). Releases and the changelog are generated from them.

## Versioning

[SemVer](https://semver.org/). Changing a role's variables in an incompatible way bumps MAJOR, a new role or feature bumps
MINOR, and a fix bumps PATCH. See [CHANGELOG.md](CHANGELOG.md).

## License

[MIT](LICENSE)
