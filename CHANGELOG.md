# Changelog

Notable changes to this role. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org/).

## Unreleased

## 1.1.0 - 2026-11-12

### Added

- `mde_linux_check_onboarding_org_id`, off by default: before installing anything, read the organisation ID inside the onboarding package on the control node and stop unless it is `mde_linux_expected_org_id`, so a package from the wrong tenant never reaches a host.
- `mde_linux_managed_config_file`: the managed configuration from a JSON or YAML file on the control node, such as an existing `mdatp_managed.json`, instead of `mde_linux_managed_config`. It works with replace, merge, removals and the `managed_config` entry point.
- A README example, "Install from package files, without Microsoft's repository": `libre_devops.standalone_package_installer` pushes `mdatp` and this role runs with `mde_linux_manage_repository: false`.
- A README example, "RHEL 8 hosts with only Python 3.6": `libre_devops.python` chained in front of this role in one play, with the requirements file for both. Requirements, Troubleshooting and the without-Galaxy guide point at it.
- `docs/using-without-galaxy.md`, linked from the README's Installation section: installing the role without Galaxy (git, release archives, requirements files that name their own source), getting its collections with no Galaxy access, keeping a copy on your own git server or artifact store, AWX, Automation Controller and execution environments, and the ansible-core and Python versions it needs.

### Changed

- `docs/using-without-galaxy.md` recommends `libre_devops.python` for managed nodes with only Python 3.6, ahead of a hand-written `raw` play and ansible-core 2.16.
- The README's Requirements and Troubleshooting explain managed nodes with only Python 3.6 (RHEL 8, SLES 15), which ansible-core 2.17 and later cannot manage, and link to the two ways round it.
- The Release workflow imports the release's own tag into Galaxy, rather than the default branch, and can be run by hand with a tag to import that tag again.
- The Release workflow imports through Galaxy's API (`scripts/galaxy-import.py`) instead of `ansible-galaxy role import`, which fails with HTTP 500 because the Galaxy namespace `libre_devops` differs from the GitHub organisation `libre-devops`. It then points the role's download links at the `libre-devops` repository and installs the tag from Galaxy to check.

### Fixed

- `ansible-galaxy role install libre_devops.mde_linux` failed with HTTP 404, because Galaxy built the 1.0.0 download link from github.com/libre_devops, which does not exist. Importing 1.0.0 again with the new Release workflow corrects it.
- `just converge` failed, because `molecule converge` has no `--platform-name`. It now runs the scenario on the one platform and keeps the container.
- The README's Galaxy badge linked to a Galaxy page that does not load without a tab; it now opens the role's documentation tab.

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
