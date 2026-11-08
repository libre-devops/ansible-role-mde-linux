# Security policy

## Reporting a vulnerability

Report vulnerabilities in this role privately, through [GitHub's private vulnerability reporting](https://github.com/libre-devops/ansible-role-mde-linux/security/advisories/new). Do not open a public issue.

Vulnerabilities in Microsoft Defender for Endpoint itself go to the [Microsoft Security Response Center](https://msrc.microsoft.com/report).

## What the role handles that is sensitive

- **Onboarding and offboarding packages.** They are specific to your tenant. Keep them encrypted with Ansible Vault or in a secret store, and never commit them. `.gitignore` blocks the usual file names. The role copies them with `no_log` and without a diff, and sets them to `0600` owned by `root`.
- **Signing keys.** The role ships Microsoft's public keys for packages.microsoft.com and imports them only when the fingerprint matches. `just check-keys` checks the bundled files in CI.
