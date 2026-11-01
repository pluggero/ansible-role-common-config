# ansible-role-common-config

[![CI](https://github.com/pluggero/ansible-role-common-config/actions/workflows/ci.yml/badge.svg)](https://github.com/pluggero/ansible-role-common-config/actions/workflows/ci.yml) [![Ansible Galaxy downloads](https://img.shields.io/ansible/role/d/pluggero/common_config?label=Galaxy%20downloads&logo=ansible&color=%23096598)](https://galaxy.ansible.com/ui/standalone/roles/pluggero/common_config)

Configure common baseline system settings — hostname, locale, keyboard layout, timezone, time sync (systemd-timesyncd), `/etc/hosts`, and DNS resolution — on Debian and Archlinux hosts.

## Requirements

**Collections:**

- `ansible.builtin` (included with Ansible)
- `community.general` (used for `pacman` package management on Archlinux and for `community.general.timezone`)

**Role dependencies:**

- None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### Hostname

```yaml
common_config_hostname: "" # leave "" to not change the hostname
```

### Locale

```yaml
common_config_locale: "en_US.UTF-8"
common_config_locale_extra: []
# - "de_DE.UTF-8"
# - "fr_FR.UTF-8"
```

`common_config_locale` is generated and set as the system default. `common_config_locale_extra` lists additional locales to generate (uncommented in `/etc/locale.gen`) without making them the default.

### Keyboard

```yaml
common_config_keyboard_layout: "us"
common_config_keyboard_variant: ""
common_config_keyboard_x11_conf: false
```

`common_config_keyboard_layout` is used both as the console keymap name (Archlinux `/etc/vconsole.conf`, Debian `/etc/default/keyboard`) and, when `common_config_keyboard_x11_conf` is `true`, as the X11 `XkbLayout` code. This works for common layouts (`us`, `de`, ...) but console keymap names and X11 layout codes are not guaranteed to match 1:1 for every layout — verify for exotic layouts.

### Timezone

```yaml
common_config_timezone: "UTC"
```

### Time sync (systemd-timesyncd)

```yaml
common_config_ntp_servers: []
# - "0.pool.ntp.org"
# - "1.pool.ntp.org"
common_config_ntp_fallback_servers: []
# - "ntp.ubuntu.com"
```

Empty lists leave `NTP=`/`FallbackNTP=` unset in `timesyncd.conf`, letting `systemd-timesyncd` fall back to its own distro defaults.

### /etc/hosts

```yaml
common_config_manage_hosts: true
common_config_hosts_entries: []
# - ip: "192.0.2.10"
#   hostnames:
#     - "app.internal"
#     - "app"
```

The rendered file always includes the standard loopback block, plus a `127.0.0.1 {{ common_config_hostname }}` line when `common_config_hostname` is set, plus one line per entry in `common_config_hosts_entries`.

### /etc/resolv.conf / systemd-resolved

```yaml
common_config_manage_resolv_conf: true
common_config_dns_servers: []
# - "1.1.1.1"
# - "9.9.9.9"
common_config_dns_fallback_servers: []
common_config_dns_search_domains: []
# - "example.internal"
```

The role detects whether `/etc/resolv.conf` is a symlink (managed by `systemd-resolved`). If it is, DNS settings are written to `/etc/systemd/resolved.conf` instead of touching the symlink. Otherwise, a static `/etc/resolv.conf` is written directly.

## Dependencies

- None

## Example Playbook

```yaml
- hosts: all
  roles:
    - role: pluggero.common-config
      vars:
        common_config_hostname: "web01"
        common_config_locale: "en_US.UTF-8"
        common_config_timezone: "Europe/Berlin"
        common_config_ntp_servers:
          - "0.pool.ntp.org"
        common_config_hosts_entries:
          - ip: "192.0.2.10"
            hostnames:
              - "db.internal"
              - "db"
        common_config_dns_servers:
          - "1.1.1.1"
          - "9.9.9.9"
```

## Testing

This role uses [Molecule](https://molecule.readthedocs.io/) with the Vagrant driver (required for exercising real `systemd` behavior):

```bash
cd molecule/default
molecule test                        # default distro: debian12
MOLECULE_DISTRO=archlinux molecule test
```

## License

MIT

## Author Information

Robin Plugge, 2026–present
