# Changelog

Notable changes to this role. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org/).

## Unreleased

### Fixed

- Galaxy import: call Galaxy's import API with the `libre_devops` namespace and the GitHub clone URL, because `ansible-galaxy role import` sends the GitHub organisation name, which Galaxy refuses with HTTP 500. The import now waits for the result and prints Galaxy's log, and the Release workflow can be run by hand to import a tag again.

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
