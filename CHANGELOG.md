# Changelog

Notable changes to this role. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org/).

## Unreleased

### Added

- `docs/using-without-galaxy.md`, linked from the README's Installation section: installing the role without Galaxy (git, release archives, requirements files that name their own source), getting its collections with no Galaxy access, keeping a copy on your own git server or artifact store, AWX, Automation Controller and execution environments, and the ansible-core and Python versions it needs.

### Changed

- `docs/using-without-galaxy.md` recommends `libre_devops.python` for managed nodes with only Python 3.6, ahead of a hand-written `raw` play and ansible-core 2.16.
- The README's Requirements and Troubleshooting explain managed nodes with only Python 3.6 (RHEL 8, SLES 15), which ansible-core 2.17 and later cannot manage, and link to the two ways round it.
- The Release workflow imports the release's own tag into Galaxy, rather than the default branch, and can be run by hand with a tag to import that tag again.

### Fixed

- `just converge` failed, because `molecule converge` has no `--platform-name`. It now runs the scenario on the one platform and keeps the container.

## 1.0.0 - 2026-11-11

### Added

- First release. Supersedes `libre-devops/mde-linux-rhel-ansible-role`, which covered RHEL only.
- `present`, `offboarded` and `absent` states.
- packages.microsoft.com repository for the RHEL, Debian, Ubuntu, SUSE, Amazon Linux and Fedora families, with both Microsoft signing keys bundled and checked by fingerprint.
- Onboarding and offboarding with the JSON packages from the Defender portal, from a file or from a variable.
- `mdatp_managed.json` in `replace` or `merge` mode, with dotted paths to delete, and a `managed_config` entry point that changes only that file.
- Licence, health, organisation and connectivity checks, and the `mde_linux_health` fact.
- Molecule scenarios on RHEL 9, Rocky 10, Debian 12 and 13, Ubuntu 24.04 and 26.04, and SLES 15 and 16.

### Changed from the RHEL-only role

- SSH key and `sshd` set-up moved out to `libre_devops.ssh_bootstrap`.
- The package update task, the quick scan and the client analyzer certificate work-around were dropped.
- Tags use the `GROUP` key only, the only key Microsoft supports in `mdatp_managed.json`.
