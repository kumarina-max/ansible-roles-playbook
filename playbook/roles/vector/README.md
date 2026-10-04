# Vector Ansible Role

Ansible role for installing and configuring [Vector](https://vector.dev/) on Debian/Ubuntu systems.

The role:

* adds the official Vector APT repository;
* installs Vector;
* creates the Vector configuration directory;
* deploys the Vector configuration from a Jinja2 template;
* validates the configuration;
* enables and starts the Vector service;
* restarts Vector when the configuration changes.

## Requirements

* Ansible 2.16 or newer
* Debian-based Linux distribution
* `systemd`
* `sudo` privileges (`become: true`)
* Internet access to the official Vector APT repository

The role currently supports:

* Ubuntu 22.04 (Jammy)
* Ubuntu 24.04 (Noble)
* Debian 11 (Bullseye)
* Debian 12 (Bookworm)

## Role Variables

### Defaults

The following variables are defined in `defaults/main.yml`:

| Variable              | Default                   | Description                    |
| --------------------- | ------------------------- | ------------------------------ |
| `vector_package_name` | `vector`                  | Vector package name            |
| `vector_config_dir`   | `/etc/vector`             | Vector configuration directory |
| `vector_config_file`  | `/etc/vector/vector.yaml` | Vector configuration file      |
| `vector_service_name` | `vector`                  | Systemd service name           |

Example:

```yaml
vector_package_name: "vector"
vector_config_dir: "/etc/vector"
vector_config_file: "/etc/vector/vector.yaml"
vector_service_name: "vector"
```

### Internal variables

The following variables are defined in `vars/main.yml` and are used internally by the role:

| Variable                      | Description                                           |
| ----------------------------- | ----------------------------------------------------- |
| `vector_architecture_map`     | Mapping between Ansible and APT architecture names    |
| `vector_package_architecture` | APT architecture detected from `ansible_architecture` |
| `vector_apt_repository`       | Official Vector APT repository                        |
| `vector_apt_key_url`          | URL of the Vector repository GPG key                  |
| `vector_apt_keyring`          | Local GPG keyring path                                |

The role automatically determines the package architecture.

## Vector Configuration

The default configuration is stored in:

```text
templates/vector.yml.j2
```

The default configuration uses Vector's `demo_logs` source and sends generated events to the console.

The template can be replaced or modified for a specific logging pipeline.

## Dependencies

This role has no dependencies on other Ansible roles.

```yaml
dependencies: []
```

## Example Playbook

```yaml
---
- name: Install and configure Vector
  hosts: vector
  become: true

  roles:
    - vector-role
```

Example inventory:

```ini
[vector]
vector-host ansible_host=192.168.1.10
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini site.yml
```

## License

MIT

## Author

kumarina-max
