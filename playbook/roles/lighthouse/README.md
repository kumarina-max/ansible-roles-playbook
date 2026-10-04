# LightHouse Ansible Role

Ansible role for installing and configuring [VKCOM LightHouse](https://github.com/VKCOM/lighthouse) as a static web application served by Nginx.

The role:

* installs Git and Nginx;
* downloads LightHouse from the official Git repository;
* creates the LightHouse installation directory;
* configures Nginx to serve LightHouse;
* enables the LightHouse Nginx site;
* removes the default Nginx site;
* validates the Nginx configuration;
* enables and starts the Nginx service;
* reloads Nginx when the configuration changes.

## Requirements

* Ansible 2.16 or newer
* Debian-based Linux distribution
* `systemd`
* `sudo` privileges (`become: true`)
* Internet access to GitHub
* Git
* Nginx

The role is designed for Debian-based Linux distributions using `apt` and `systemd`.

The role was tested successfully on:

* Linux Mint 22.2

## Role Variables

The following variables are defined in `defaults/main.yml`.

| Variable                          | Default                                   | Description                            |
| --------------------------------- | ----------------------------------------- | -------------------------------------- |
| `lighthouse_repo`                 | `https://github.com/VKCOM/lighthouse.git` | LightHouse Git repository              |
| `lighthouse_version`              | `master`                                  | Git branch, tag or commit to deploy    |
| `lighthouse_install_dir`          | `/var/www/lighthouse`                     | LightHouse installation directory      |
| `lighthouse_nginx_server_name`    | `_`                                       | Nginx `server_name`                    |
| `lighthouse_nginx_port`           | `8080`                                    | Port used by Nginx to serve LightHouse |
| `lighthouse_nginx_config`         | `/etc/nginx/sites-available/lighthouse`   | Nginx site configuration               |
| `lighthouse_nginx_enabled_config` | `/etc/nginx/sites-enabled/lighthouse`     | Enabled Nginx site                     |
| `lighthouse_nginx_service`        | `nginx`                                   | Nginx systemd service name             |

### Example variables

```yaml
lighthouse_repo: "https://github.com/VKCOM/lighthouse.git"
lighthouse_version: "master"

lighthouse_install_dir: "/var/www/lighthouse"

lighthouse_nginx_server_name: "lighthouse.example.com"
lighthouse_nginx_port: 8080

lighthouse_nginx_config: "/etc/nginx/sites-available/lighthouse"
lighthouse_nginx_enabled_config: "/etc/nginx/sites-enabled/lighthouse"

lighthouse_nginx_service: "nginx"
```

## Internal Variables

The role uses the following internal variable from `vars/main.yml`:

| Variable                          | Description                                  |
| --------------------------------- | -------------------------------------------- |
| `lighthouse_nginx_default_config` | Path to the default Nginx site configuration |

Default value:

```yaml
lighthouse_nginx_default_config: "/etc/nginx/sites-enabled/default"
```

## LightHouse Configuration

LightHouse is a static web application served directly by Nginx.

The role does not configure ClickHouse connection parameters because the LightHouse web application itself provides the ClickHouse connection settings.

The Nginx configuration template is stored in:

```text
templates/lighthouse.conf.j2
```

Example generated configuration:

```nginx
server {
    listen 8080;
    listen [::]:8080;

    server_name _;

    root /var/www/lighthouse;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## Dependencies

This role has no dependencies on other Ansible roles.

```yaml
dependencies: []
```

## Example Playbook

```yaml
---
- name: Install and configure LightHouse
  hosts: lighthouse
  become: true

  roles:
    - lighthouse-role
```

Example inventory:

```ini
[lighthouse]
lighthouse-host ansible_host=192.168.1.10
```

Run the playbook:

```bash
ansible-playbook -i inventory.ini site.yml
```

After installation, LightHouse will be available at:

```text
http://192.168.1.10:8080
```

## Testing

The role contains a test playbook:

```text
tests/test.yml
```

Syntax check:

```bash
ANSIBLE_ROLES_PATH=~/netology-ansible-roles \
ansible-playbook \
  -i tests/inventory \
  tests/test.yml \
  --syntax-check
```

Run the role:

```bash
ANSIBLE_ROLES_PATH=~/netology-ansible-roles \
ansible-playbook \
  -i tests/inventory \
  tests/test.yml \
  --ask-become-pass
```

The role was tested successfully on Linux Mint 22.2.

The second execution completed with:

```text
changed=0
failed=0
```

which confirms idempotent behaviour.

## License

MIT

## Author

kumarina-max
