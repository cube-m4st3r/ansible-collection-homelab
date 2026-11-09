---
name: semver
description: >-
  Versioning and commit-message rules for the ansible-collection-homelab repository (Ansible collection cube_m4st3r.homelab). Use when writing or
  reviewing a commit message or PR title, deciding whether a change is breaking, or predicting
  the next release version.
---

# Semantic versioning for `ansible-collection-homelab`

This repo is the Ansible collection `cube_m4st3r.homelab`, published to Ansible Galaxy.
Its users are people who put its roles in their playbooks, so its public API is: the role
names, every role variable (documented in `meta/argument_specs.yml`), their defaults, and the
files, units and paths the roles create on managed hosts.

## How versions are made here

- Versions are **never chosen by hand**. release-please reads the Conventional Commit messages on
  `main`, keeps a release PR open with the next version and the generated `CHANGELOG.md`, and
  merging that PR creates the tag (e.g. `v1.4.0`) and triggers the Galaxy publish workflow.
- Never edit `version:` in `galaxy.yml`, `.release-please-manifest.json`, `CHANGELOG.md`, or create git tags yourself. Your job is to write
  commit messages that make release-please compute the right version.
- If a pull request is squash-merged, its title becomes the commit message, so PR titles follow
  the same format.
- Versions follow SemVer `MAJOR.MINOR.PATCH`. While the latest release is still `0.x`, a breaking
  change is marked exactly the same way; release-please (configured with `bump-minor-pre-major`)
  turns it into a MINOR bump until `1.0.0`.

## Commit message format

```text
<type>(<scope>)<!>: <summary>

<body: why the change was made, not what the diff shows>

BREAKING CHANGE: <what breaks, and the exact steps to migrate>
```

- `summary`: imperative mood, lowercase start, no trailing period, at most 72 characters.
- `scope` (optional but preferred): the role name (`podman_host`, `quadlet_app`, `authentik`, ...), or `galaxy`, `molecule`,
  `ci`, `deps`.
- `!` after the type/scope **and** a `BREAKING CHANGE:` footer for every breaking change. The footer
  is what users read in the changelog, so it must explain the migration, not just repeat the summary.

## Type → version bump

| Type | Use for | Bump |
| --- | --- | --- |
| `feat` | Something new a user of this repo can use | MINOR |
| `fix` | A bug fix users would notice | PATCH |
| `perf` | Faster or lighter, same behaviour | PATCH |
| `fix(deps)` | Raising a dependency (e.g. `containers.podman`) for a fix users need | PATCH |
| `docs`, `refactor`, `test`, `ci`, `build`, `style`, `chore` | Everything with no effect on what users get | none |
| any type + `!` / `BREAKING CHANGE:` | See the checklist below | MAJOR |

Pick the type by the **effect on users**, not by the size of the diff.

## Breaking change checklist for `ansible-collection-homelab`

A change is breaking (`!` + footer) if **any** of these is true:

- A role is removed or renamed.
- A role variable is removed, renamed, or its type changes (also when the argument spec gets stricter).
- A default changes in a way that alters existing deployments, e.g. rootless → rootful, a different
  service user, a different data or volume path, different published ports.
- Generated Quadlet file names or systemd unit names change. Existing services would be orphaned
  or restarted under a new name.
- The minimum supported ansible-core, Python or Podman version goes up, or a host OS is dropped.
- A dependency's minimum version goes up in a way that forces users to upgrade it themselves.
- Behaviour that users rely on changes without a variable change, e.g. `quadlet_app` starts
  removing apps it did not remove before.

## MINOR in this repo

- A new role.
- A new optional variable whose default keeps the current behaviour.
- A new supported OS or Podman feature.

## PATCH in this repo

- A bug fix in a role that does not change its interface.
- An idempotence fix (the second run no longer reports changes).
- `fix(deps)`: raising a dependency to pick up a fix, without forcing users to act.

## No release in this repo

- README, role docs, ADR references: `docs`.
- Molecule, CI, linting, pre-commit: `test`, `ci`, `chore`.
- Refactoring tasks with identical results: `refactor`.

## Procedure

1. Look at the change: `git diff --staged` (or the PR diff).
2. Go through the breaking checklist above, item by item. If one matches, use `!` and write a
   `BREAKING CHANGE:` footer with migration steps.
3. When it is unclear whether something breaks users, **ask** instead of guessing. Hiding a break
   in a `feat` or `fix` is worse than flagging one too many.
4. Pick the type and scope by user-visible effect.
5. One logical change per commit. If the diff mixes unrelated changes (e.g. a feature and an
   unrelated fix), suggest splitting it into separate commits.
6. Write the message. Keep the body short and focused on *why*.

## Predicting the next version

```bash
last=$(git describe --tags --abbrev=0)
git log "$last"..HEAD --format='%s%n%b'
```

The highest bump among those commits wins: any breaking change → MAJOR (MINOR while `0.x`),
else any `feat` → MINOR, else any `fix`/`perf`, `fix(deps)` → PATCH, else no release.

## Examples

```text
feat(quadlet_app): support named volumes via quadlet_apps[].volumes
```

```text
fix(podman_host): enable lingering before starting user services

Services stopped after logout because linger was enabled after the
first start.
```

```text
feat(quadlet_app)!: rename quadlet_apps[].ports to publish

Matches the Quadlet key PublishPort and frees "ports" for future use.

BREAKING CHANGE: rename `ports` to `publish` in every entry of
`quadlet_apps`. The values keep the same format.
```
