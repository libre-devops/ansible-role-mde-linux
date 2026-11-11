<div align="center">
  <a href="https://libredevops.org">
    <picture>
      <source media="(prefers-color-scheme: dark)" srcset="https://libredevops.org/assets/libre-devops-white.png">
      <img alt="Libre DevOps" src="https://libredevops.org/assets/libre-devops-black.png" width="300">
    </picture>
  </a>
</div>

# Ansible Role: Microsoft Defender for Endpoint on Linux

Installs, configures, onboards and offboards Microsoft Defender for Endpoint (`mdatp`) on Linux servers, from Microsoft's signed packages.microsoft.com repository.

[![CI](https://github.com/libre-devops/ansible-role-mde-linux/actions/workflows/ci.yml/badge.svg)](https://github.com/libre-devops/ansible-role-mde-linux/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/libre-devops/ansible-role-mde-linux?sort=semver&label=release)](https://github.com/libre-devops/ansible-role-mde-linux/releases/latest)
[![Galaxy](https://img.shields.io/badge/galaxy-libre__devops.mde__linux-EE0000?logo=ansible&logoColor=white)](https://galaxy.ansible.com/ui/standalone/roles/libre_devops/mde_linux/)
[![Licence](https://img.shields.io/github/license/libre-devops/ansible-role-mde-linux)](./LICENSE)

---

## What it does

| State | What happens |
|---|---|
| `present` (default) | Adds the packages.microsoft.com repository and Microsoft's signing keys, installs `mdatp`, writes `mdatp_managed.json` if you give settings, onboards the device and checks it is licensed, healthy and connected. |
| `offboarded` | Offboards the device with the offboarding package and leaves the agent installed. |
| `absent` | Offboards the device (when you give an offboarding package), then removes the agent. |

It also has a `managed_config` entry point that changes only `mdatp_managed.json`, for when the agent is already in place. See [Managed configuration](#managed-configuration).

Microsoft recommends its own deployment tool or installer script for most estates, and documents Ansible as a supported route ([Microsoft Learn: Deploy with Ansible](https://learn.microsoft.com/en-us/defender-endpoint/linux-install-with-ansible)). This role follows the "configure repositories manually" method on that page, with the onboarding and offboarding steps taken from Microsoft's `mde_installer.sh`. Microsoft does not support third-party tools such as Ansible, so raise problems with the role here, not with Microsoft.

## Contents

- [Requirements](#requirements)
- [Supported platforms](#supported-platforms)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Getting the onboarding and offboarding packages](#getting-the-onboarding-and-offboarding-packages)
- [Role variables](#role-variables)
- [Managed configuration](#managed-configuration)
- [Repository and signing keys](#repository-and-signing-keys)
- [Checks after onboarding](#checks-after-onboarding)
- [Examples](#examples)
- [Troubleshooting](#troubleshooting)
- [Developing](#developing)
- [Releasing](#releasing)
- [Sources](#sources)

## Requirements

| What | Detail |
|---|---|
| Control node | `ansible-core` 2.16 or later. |
| Collections | `community.general`, for SUSE hosts only (`ansible.builtin.package` hands over to `community.general.zypper` there). The `ansible` package already includes it. |
| Managed nodes | A [supported distribution](#supported-platforms) with systemd, on x86_64 or aarch64, and a Python that your `ansible-core` supports. RHEL 8 and SLES 15 ship Python 3.6, which ansible-core 2.17 and later cannot use: see [Python on managed nodes](./docs/using-without-galaxy.md#on-managed-nodes). |
| Privileges | `become: true`. The role installs packages and writes under `/etc`. |
| Network | HTTPS to `packages.microsoft.com` (or your mirror) and to the Defender for Endpoint service URLs ([Microsoft Learn: Configure your network environment](https://learn.microsoft.com/en-us/defender-endpoint/configure-environment)). |
| Licence | Defender for Endpoint (or Defender for Servers) licences, and the onboarding package from your tenant. |

Microsoft's own minimums are 1 CPU core, 1 GB of RAM and 2 GB of disk ([Microsoft Learn: Prerequisites](https://learn.microsoft.com/en-us/defender-endpoint/mde-linux-prerequisites)).

## Supported platforms

The role works out the packages.microsoft.com path from the host's facts, with the same mapping as Microsoft's `mde_installer.sh`. CI installs and configures the agent on every platform in the "Tested in CI" column on each pull request.

| Family | Distributions | Repository path | Tested in CI |
|---|---|---|---|
| RHEL | RHEL, Rocky, Alma, Oracle Linux, CentOS Stream 8, 9 and 10 | `rhel/<major>` | RHEL 9 (UBI), Rocky 10 |
| Amazon | Amazon Linux 2023 and 2 | `amazonlinux/<major>` | No |
| Fedora | Fedora 43 and later (earlier releases) | `fedora/<version>` (`rhel/8`) | No |
| Debian | Debian 12 and 13 | `debian/<major>` | Debian 12, Debian 13 |
| Ubuntu | Ubuntu 22.04, 24.04 and 26.04 LTS | `ubuntu/<version>` | Ubuntu 24.04, Ubuntu 26.04 |
| SUSE | SLES 15 and 16, openSUSE Leap 15 and 16 | `sles/<major>` | SLES 15, SLES 16 (BCI) |

Microsoft's full list, with the exact minor versions and kernel requirements, is in [Microsoft Learn: Supported Linux distributions](https://learn.microsoft.com/en-us/defender-endpoint/mde-linux-prerequisites#supported-linux-distributions). A distribution missing from that list is not supported by Microsoft, whatever this role does.

The tests run in containers, where the agent installs but cannot license itself. Onboarding, offboarding and the health checks need a real VM: see [Developing](#developing).

## Installation

From Galaxy:

```bash
ansible-galaxy role install libre_devops.mde_linux
```

Or pin a release from GitHub in a `requirements.yml`, as in [`examples/requirements.yml`](./examples/requirements.yml):

```yaml
roles:
  - name: libre_devops.mde_linux
    src: https://github.com/libre-devops/ansible-role-mde-linux.git
    scm: git
    version: 1.0.0
```

Without Galaxy, or to keep a copy inside your organisation, see [Using the role without Galaxy](./docs/using-without-galaxy.md). It covers git, release archives, requirements files that point at your own servers, AWX and Automation Controller, execution environments, and managed nodes with Python 3.6 such as RHEL 8.

## Quick start

```yaml
- name: Defender for Endpoint on Linux servers
  hosts: linux_servers
  become: true
  roles:
    - role: libre_devops.mde_linux
      vars:
        mde_linux_onboarding_file: files/mdatp_onboard.json   # encrypted with ansible-vault
        mde_linux_expected_org_id: 00000000-0000-0000-0000-000000000000
```

```bash
ansible-playbook -i inventory.ini site.yml --ask-vault-pass
```

The first run installs the agent, onboards the device and waits for it to report licensed and healthy. Later runs change nothing unless a setting changed. The device shows in the Defender portal's device inventory after 5 to 20 minutes.

## Getting the onboarding and offboarding packages

Both are specific to your tenant. Treat them as secrets.

1. In the [Microsoft Defender portal](https://security.microsoft.com), go to **System** > **Settings** > **Endpoints** > **Device management** > **Onboarding** (or **Offboarding**).
2. Select **Linux Server** as the operating system and **Your preferred Linux configuration management tool** as the deployment method.
3. Download the package and extract it. Onboarding gives `mdatp_onboard.json`. Offboarding gives `mdatp_offboard.json`, and the zip name carries its expiry date.
4. Encrypt the file before it goes anywhere near a repository:

   ```bash
   ansible-vault encrypt files/mdatp_onboard.json
   ```

Pass the file with `mde_linux_onboarding_file` (Ansible decrypts vaulted files as it copies them), or pass the content with `mde_linux_onboarding_content`, for example from a secret store lookup. The offboarding package works the same way through `mde_linux_offboarding_file` or `mde_linux_offboarding_content`.

The role only takes the JSON packages. The "Local script" packages (`MicrosoftDefenderATPOnboardingLinuxServer.py` and the offboarding script) are for running by hand.

To find your organisation ID for `mde_linux_expected_org_id`, run `mdatp health --field org_id` on a device that is already onboarded.

## Role variables

Every variable is validated against [`meta/argument_specs.yml`](./meta/argument_specs.yml) when the role starts, so a typo in a choice fails straight away. Defaults are in [`defaults/main.yml`](./defaults/main.yml).

### State, channel and package

| Variable | Default | Description |
|---|---|---|
| `mde_linux_state` | `present` | `present`, `offboarded` or `absent`. See [What it does](#what-it-does). |
| `mde_linux_channel` | `prod` | `prod`, `insiders-slow` or `insiders-fast`. Microsoft does not support switching channel in place: run with `absent`, then `present` on the new channel. |
| `mde_linux_package_state` | `present` | `present` installs if missing. `latest` also upgrades. |
| `mde_linux_version` | `""` | Exact version to install, for example `101.25092.0005`. Empty takes what the repository offers. Insiders builds have a suffix, so copy the version as the package manager lists it. Cannot be combined with `latest`. |

### Repository

| Variable | Default | Description |
|---|---|---|
| `mde_linux_manage_repository` | `true` | Add the repository and keys. Set `false` when the host already gets `mdatp` from somewhere else, such as Red Hat Satellite or an internal mirror. |
| `mde_linux_repository_url` | `https://packages.microsoft.com` | Base URL. Point it at a mirror with the same layout if you have one. |
| `mde_linux_repository_path` | `""` | Path under the base URL, such as `rhel/9`. Empty works it out from the host's facts. |
| `mde_linux_remove_repository` | `false` | With `absent`, also remove the repository definition. Off by default because other Microsoft agents (such as the Azure Arc agent) use the same one. |

### Onboarding and offboarding

| Variable | Default | Description |
|---|---|---|
| `mde_linux_onboard` | `true` | Onboard the device. Set `false` to install the agent only, for example in a golden image. |
| `mde_linux_onboarding_file` | `""` | Path on the control node to `mdatp_onboard.json`. Can be vaulted. |
| `mde_linux_onboarding_content` | `""` | The JSON itself. Use instead of the file. |
| `mde_linux_expected_org_id` | `""` | Organisation the device must belong to. If it is onboarded to another one, the role stops rather than overwrite it. Empty skips the check. |
| `mde_linux_force_onboarding` | `false` | Copy the onboarding package even when the device is already licensed. |
| `mde_linux_offboarding_file` | `""` | Path on the control node to `mdatp_offboard.json`. Can be vaulted. Required for `offboarded`. |
| `mde_linux_offboarding_content` | `""` | The JSON itself. Use instead of the file. |

### Managed configuration

| Variable | Default | Description |
|---|---|---|
| `mde_linux_managed_config` | `{}` | Settings for `mdatp_managed.json`, in [Microsoft's schema](https://learn.microsoft.com/en-us/defender-endpoint/linux-preferences). |
| `mde_linux_managed_config_mode` | `replace` | `replace` writes the settings as the whole file. `merge` deep-merges them into the file already on the host. |
| `mde_linux_managed_config_list_merge` | `replace` | How `merge` treats lists, as the `list_merge` option of Ansible's `combine` filter: `replace`, `keep`, `append`, `prepend`, `append_rp` or `prepend_rp`. |
| `mde_linux_managed_config_remove` | `[]` | Dotted paths to delete afterwards, such as `cloudService.proxy`. |
| `mde_linux_restart_on_managed_config_change` | `false` | Restart `mdatp` when the file changes. Only the diagnostic level and log rotation settings need a restart. |

With `mde_linux_managed_config` and `mde_linux_managed_config_remove` both empty, the role leaves `mdatp_managed.json` alone.

### Checks and clean-up

| Variable | Default | Description |
|---|---|---|
| `mde_linux_verify` | `true` | After onboarding, wait for the device to be licensed and healthy, and check the organisation. |
| `mde_linux_verify_connectivity` | `true` | Also run `mdatp connectivity test`. |
| `mde_linux_health_retries` | `18` | How many times to poll `mdatp health`. |
| `mde_linux_health_delay` | `10` | Seconds between polls. The defaults wait up to 3 minutes. |
| `mde_linux_remove_data` | `false` | With `absent`, also delete `/etc/opt/microsoft/mdatp`, `/var/opt/microsoft/mdatp` and `/var/log/microsoft/mdatp`. |

### Facts set by the role

| Fact | Description |
|---|---|
| `mde_linux_health` | The parsed output of `mdatp health --output json` after the checks, for use by later tasks. Empty when the checks are off. |

## Managed configuration

`mdatp_managed.json` in `/etc/opt/microsoft/mdatp/managed/` is the agent's local policy. Settings in it take precedence over local settings, and exclusions added through it can only be removed through it ([Microsoft Learn: Set preferences](https://learn.microsoft.com/en-us/defender-endpoint/linux-preferences)). If you manage the agent through Security Settings Management in the Defender portal instead, leave `mde_linux_managed_config` empty.

### Replace

The default. The variable is the whole file:

```yaml
mde_linux_managed_config:
  antivirusEngine:
    enforcementLevel: real_time
    threatTypeSettings:
      - key: potentially_unwanted_application
        value: block
      - key: archive_bomb
        value: audit
  cloudService:
    enabled: true
    automaticDefinitionUpdateEnabled: true
    automaticSampleSubmissionConsent: safe
  edr:
    tags:
      - key: GROUP
        value: web-servers
```

This is Microsoft's recommended starter profile plus a device tag. `GROUP` is the only tag key Microsoft supports in this file, and the role fails on any other key.

### Merge and remove

`merge` keeps whatever is already in the file, for example settings another team or tool wrote, and changes only what you give. Deleting a key needs `mde_linux_managed_config_remove`, because a merge can only add or change.

```yaml
mde_linux_managed_config_mode: merge
mde_linux_managed_config_list_merge: append_rp   # add to lists, keep what is there
mde_linux_managed_config:
  antivirusEngine:
    exclusions:
      - $type: excludedPath
        isDirectory: true
        path: /var/lib/postgresql
mde_linux_managed_config_remove:
  - cloudService.proxy
```

With `append_rp`, running this again changes nothing, because the exclusion is already there. To drop an exclusion, give the full list you want with `list_merge: replace`.

### Only the managed configuration

On hosts where the agent is already installed and onboarded, change just the profile:

```yaml
- name: Add an exclusion on the database servers
  hosts: db_servers
  become: true
  tasks:
    - name: Merge the exclusion into mdatp_managed.json
      ansible.builtin.include_role:
        name: libre_devops.mde_linux
        tasks_from: managed_config
      vars:
        mde_linux_managed_config_mode: merge
        mde_linux_managed_config_list_merge: append_rp
        mde_linux_managed_config:
          antivirusEngine:
            exclusions:
              - $type: excludedPath
                isDirectory: true
                path: /var/lib/postgresql
```

The entry point has its own argument specification, so it checks the same variables.

### Restarts

Most settings apply without restarting `mdatp`. Microsoft lists the diagnostic level and the log rotation settings as needing a restart. Set `mde_linux_restart_on_managed_config_change: true` when you change those.

## Repository and signing keys

The role writes the same repository definitions as Microsoft's `packages-microsoft-prod` package, with the same file names and repository IDs. A host set up by hand, by this role, or by another role that uses packages.microsoft.com (such as `libre_devops.azure_arc`) ends up with one definition, not two that conflict.

| Package manager | Files written |
|---|---|
| dnf or yum | `/etc/pki/rpm-gpg/microsoft.asc`, `/etc/pki/rpm-gpg/microsoft-2025.asc`, `/etc/yum.repos.d/microsoft-<channel>.repo` (repository ID `packages-microsoft-com-<channel>`) |
| zypper | The same keys, and `/etc/zypp/repos.d/microsoft-<channel>.repo` |
| apt | `/usr/share/keyrings/microsoft-prod.gpg` and `/etc/apt/sources.list.d/microsoft-<channel>.list` |

Microsoft signs older repositories with its original key and newer ones (RHEL 10, SLES 16, Debian 13, Ubuntu 26.04, Fedora 43 and later) with its 2025 key. The role installs both, so one definition works on every distribution:

| Key | Fingerprint |
|---|---|
| `microsoft.asc` | `BC52 8686 B50D 79E3 39D3 721C EB3E 94AD BE12 29CF` |
| `microsoft-2025.asc` | `AA86 F75E 427A 19DD 3334 6403 EE4D 7792 F748 182B` |

The keys ship with the role, so nothing is downloaded on trust at run time. On RPM systems `rpm_key` imports each key only if its fingerprint matches. Package and repository metadata signature checks (`gpgcheck`, `repo_gpgcheck`) stay on. `just check-keys` checks the bundled files in CI. The fingerprints are published on [Microsoft Learn: Linux package repository](https://learn.microsoft.com/en-us/linux/packages).

With a channel other than `prod`, the role adds that channel's repository. If another role has also added the `prod` repository, the package manager installs the newest `mdatp` it can see, which is normally the Insiders one.

## Checks after onboarding

With `mde_linux_verify: true` (the default) the role:

1. Waits for `mdatp health --field licensed` to return `true`.
2. Waits for `mdatp health --field healthy` to return `true`.
3. Checks `org_id` matches `mde_linux_expected_org_id`, if you set it.
4. Runs `mdatp connectivity test`, if `mde_linux_verify_connectivity` is `true`.
5. Sets the `mde_linux_health` fact from `mdatp health --output json`.

These are the checks in Microsoft's post-installation playbook, without the EICAR test file download.

When it is first onboarded, the agent downloads the latest security intelligence, which can take a few minutes on a slow link. Raise `mde_linux_health_retries` if the checks time out on first run.

## Examples

### Install only, for a golden image

```yaml
mde_linux_onboard: false
```

Microsoft's golden image guidance says to copy the onboarding file only while the `mdatp` service is stopped ([Microsoft Learn: Golden images](https://learn.microsoft.com/en-us/defender-endpoint/linux-deploy-defender-for-endpoint-using-golden-images)). Onboard each clone after it boots, with a second run of the role.

### Passive mode alongside another antivirus product

```yaml
mde_linux_managed_config:
  antivirusEngine:
    enforcementLevel: passive
```

Passive mode keeps full EDR visibility without real-time protection. Run scheduled scans in this mode.

### Pin a version from a local mirror

```yaml
mde_linux_repository_url: https://mirror.example.com/packages.microsoft.com
mde_linux_version: 101.25092.0005
```

### Offboard but keep the agent

```yaml
mde_linux_state: offboarded
mde_linux_offboarding_file: files/mdatp_offboard.json   # vaulted, check the expiry date
```

Microsoft suggests tagging a device before offboarding it, because the portal does not label offboarded devices.

### Remove the agent completely

```yaml
mde_linux_state: absent
mde_linux_offboarding_file: files/mdatp_offboard.json
mde_linux_remove_data: true
```

Without an offboarding package the agent is removed without offboarding, and the device turns inactive in the portal after seven days. Either way, the device stays in the inventory for up to 180 days.

## Troubleshooting

| Symptom | What to check |
|---|---|
| `The module interpreter '/usr/bin/python3' was not found` or `Module result deserialization failed`, on the first task | The host only has Python 3.6 (RHEL 8, SLES 15), which ansible-core 2.17 and later cannot use. Every module fails, not only this role's. See [Python on managed nodes](./docs/using-without-galaxy.md#on-managed-nodes). |
| `Cannot determine license status` | The `mdatp` daemon is not running. Check `systemctl status mdatp` and `/var/log/microsoft/mdatp/`. |
| The licence check times out | The onboarding package is from the wrong tenant or out of date, or the host cannot reach the Defender URLs. Run `mdatp health --field health_issues` and `mdatp connectivity test` on the host. |
| `This device is onboarded to organisation ...` | The device belongs to another tenant. Offboard it with that tenant's package, then run the role again. |
| `healthy` stays `false` | Run `mdatp health` and read `health_issues`. [Microsoft Learn: Health status](https://learn.microsoft.com/en-us/defender-endpoint/health-status) explains each one. |
| apt reports `Conflicting values set for option Signed-By` | Another file in `/etc/apt/sources.list.d` points at packages.microsoft.com with a different keyring. Remove it, or set `mde_linux_manage_repository: false` and keep yours. |
| `wdavdaemon: error while loading shared libraries: libcap.so.2` (Ubuntu 26.04) | The `mdatp` package needs `libcap2` without declaring it. It is on every standard Ubuntu install, but not on minimal images: `apt install libcap2`. |
| `%prein(mdatp...) scriptlet failed` with `logger: command not found` (SLES 16) | The package's scripts need `logger` without declaring it. Install `util-linux-systemd`. |
| Ansible fails on SLES 15 with a Python version error | The host is on Python 3.6. Install `python311` and set `ansible_python_interpreter: /usr/bin/python3.11`. |
| A setting in `mdatp_managed.json` has no effect | Run `mdatp health`. Managed settings show `[managed]` next to them. The diagnostic level and log rotation settings need `mde_linux_restart_on_managed_config_change: true`. |

## Developing

You need [uv](https://docs.astral.sh/uv/), [just](https://github.com/casey/just) and [podman](https://podman.io/). CI runs the same recipes.

| Recipe | What it does |
|---|---|
| `just deps` | Installs the pinned Python tools (`uv.lock`) and the collections in `requirements.yml` into `.ansible/`. |
| `just lint` | `yamllint`, `ansible-lint` on the production profile, and `zizmor` on the workflows. |
| `just test [scenario] [platform]` | Runs a Molecule scenario on one platform, or on all of them. `just test default debian13`. |
| `just test-all` | Every scenario on every platform, one after another. |
| `just converge <scenario> <platform>` | Runs a scenario on one platform and leaves the container running, pass or fail. `just login` opens a shell in it, `just destroy` removes it. |
| `just live <inventory> [args]` | Runs [`examples/playbook.yml`](./examples/playbook.yml) against real hosts, checks included. |
| `just check-keys` | Checks the bundled keys match the fingerprints the role trusts. |
| `just secrets` | Scans the git history with gitleaks. |
| `just platforms`, `just scenarios` | List what can be tested. |

### Molecule scenarios

| Scenario | What it proves |
|---|---|
| `default` | Repository, keys, package, service, `mdatp_managed.json` and the onboarding file, then a second run with no changes. |
| `remove` | `absent` with an offboarding package, `mde_linux_remove_repository` and `mde_linux_remove_data` leave nothing behind. |
| `managed_config` | The entry point merges into an existing profile, deletes a key, and changes nothing on a second run. |

Every platform runs systemd as PID 1 in a podman container, built from the vendor's image with `sudo`, Python, and the few packages a normal server install has and the minimal images lack (`libcap2`, `util-linux-systemd`). The RHEL 10 family runs on Rocky 10, because the UBI 10 repositories do not carry the `iptables` package `mdatp` needs. Container names start with `mde-`, so tests of the other Libre DevOps roles can run alongside. The agent cannot license itself in a container, so the scenarios use dummy onboarding and offboarding packages from `molecule/fixtures` and switch the health checks off. Test onboarding, offboarding and the checks against a lab VM onboarded to a test tenant with `just live`.

## Releasing

1. Move the Unreleased entries in [`CHANGELOG.md`](./CHANGELOG.md) under the new version.
2. Run `just release 1.2.3`. It tags the commit and pushes the tag.
3. The Release workflow creates the GitHub release and imports the tag into Galaxy, which takes the version from the tag. It needs the `GALAXY_API_KEY` secret, which the libre-devops organisation provides. If the import fails, fix the cause and run the Release workflow by hand with the tag to import it again.

## Sources

Microsoft Learn pages this role is built from, retrieved 8 November 2026.

| Topic | Page |
|---|---|
| Deploying with Ansible | [Deploy Microsoft Defender for Endpoint on Linux with Ansible](https://learn.microsoft.com/en-us/defender-endpoint/linux-install-with-ansible) |
| Prerequisites and supported distributions | [Prerequisites for Microsoft Defender for Endpoint on Linux](https://learn.microsoft.com/en-us/defender-endpoint/mde-linux-prerequisites) |
| Channels, manual install and uninstall | [Deploy Microsoft Defender for Endpoint on Linux manually](https://learn.microsoft.com/en-us/defender-endpoint/linux-install-manually) |
| Managed configuration profile | [Set preferences for Microsoft Defender for Endpoint on Linux](https://learn.microsoft.com/en-us/defender-endpoint/linux-preferences) |
| Offboarding | [Offboard Microsoft Defender for Endpoint on Linux](https://learn.microsoft.com/en-us/defender-endpoint/linux-off-board-endpoints) |
| Health fields | [Defender for Endpoint on Linux health status](https://learn.microsoft.com/en-us/defender-endpoint/health-status) |
| Signing keys | [Linux package repository for Microsoft software](https://learn.microsoft.com/en-us/linux/packages) |
| Installer script logic | [`mde_installer.sh` in microsoft/mdatp-xplat](https://github.com/microsoft/mdatp-xplat/blob/master/linux/installation/mde_installer.sh) |

## Licence

[MIT](./LICENSE).
