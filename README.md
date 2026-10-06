# Ansible Role: update

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-update)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-update)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-update)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-update/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-update/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-update/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-update/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for updating installed system packages and optionally rebooting
when required.

## Purpose

Install all available updates for installed system packages on AlmaLinux,
Debian, Fedora, openSUSE Leap, openSUSE Tumbleweed, and Ubuntu. Reboots are
disabled by default and can be enabled with update_reboot.

## Scope

### Managed

- Package metadata refresh and system updates through APT, DNF/DNF5, or Zypper.
- Optional reboot detection, immediate reboot with connection recovery, or a
  scheduled reboot.

### Not Managed

- Repository configuration, release selection, package holds, and scheduled
  updates.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `update_reboot`

Type: `bool`. Required: `false`.

Reboot after updating packages when the platform reports that a reboot is
needed.
Pending reboot requests from earlier updates are also honored.

Default:

```yaml
update_reboot: false
```

### `update_reboot_time`

Type: `str`. Required: `false`.

Reboot immediately with now, or schedule the next occurrence of a local 24-hour
HH:MM time.
Only applies when update_reboot is true and a reboot is needed.
Scheduled reboots require systemd and return without waiting for the reboot.

Default:

```yaml
update_reboot_time: now
```

## Check Mode

Package modules report pending changes without installing updates.

- Reboot detection and reboot helper installation are skipped in check mode;
  reboot requirements are not predicted.
- Package managers use their available cached metadata in check mode.

## Service Behavior

Package maintainer scripts may restart services during updates. With
update_reboot enabled, the role reboots only when the platform reports a pending
reboot. Immediate reboots wait for the host to become reachable again; scheduled
reboots return after registering the shutdown request.

## Operational Notes

- APT uses dist-upgrade; DNF and DNF5 update installed packages; Zypper uses
  dist-upgrade to synchronize packages with the enabled repositories, including
  Tumbleweed snapshots. Dependency resolution can install or remove packages.
  Repository URLs and release versions are not changed.
- APT metadata is cached for one hour. Repeated runs are unchanged when no
  further package updates or reboot requests are pending, apart from metadata
  refreshes reported by the package manager.
- With update_reboot enabled, Debian and Ubuntu install needrestart and check
  pending kernel or CPU microcode updates as well as /run/reboot-required.
  Unknown needrestart results alone do not request a reboot.
- AlmaLinux and Fedora install the matching DNF needs-restarting plugin and use
  its reboot hint. openSUSE uses zypper needs-rebooting. Detection runs after
  updates, including when a previous run left a reboot pending.
- Enable reboots on machines with an independent boot lifecycle. Containers
  share their host kernel and do not provide reliable host reboot detection.
- Use serial in the calling play to update and reboot hosts in batches.
- update_reboot_time accepts now or a quoted 24-hour time such as '23:30'.
  Scheduled reboots use the target host's local timezone and the next occurrence
  of that time, tomorrow if the time has already passed.
- Scheduling requires systemd-logind. An identical pending reboot schedule is
  retained; a different pending shutdown schedule is replaced. Disabling
  update_reboot does not cancel a previously scheduled reboot. Use shutdown -c
  on the target host to cancel a pending shutdown explicitly.
- serial controls update batches and immediate reboots; hosts scheduled for the
  same local time may reboot together. Scheduled shutdowns do not survive an
  intervening reboot.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Update all packages without rebooting

Install available updates with the default reboot policy.

```yaml
---
- name: Update system packages
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.update
```

### Reboot when required

Update hosts one at a time and reboot hosts with a pending reboot.

```yaml
---
- name: Update systems with optional reboots
  hosts: all
  gather_facts: true
  serial: 1
  roles:
    - role: jomrr.update
      update_reboot: true
```

### Schedule a required reboot

Install updates now and schedule any required reboot for the next local 23:30.

```yaml
---
- name: Update systems and schedule reboots
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.update
      update_reboot: true
      update_reboot_time: '23:30'
```

## References

- [APT module](https://docs.ansible.com/projects/ansible/latest/collections/ansible/builtin/apt_module.html)
- [Zypper manual](https://manpages.opensuse.org/Tumbleweed/zypper/zypper.8.en.html)
- [needrestart batch output](https://github.com/liske/needrestart/blob/master/README.batch.md)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2026 Jonas Mauer.
